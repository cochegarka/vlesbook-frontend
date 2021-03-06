<script>
    import 'bulma/css/bulma.css'
    import '@fortawesome/fontawesome-free/css/all.css'
    import {fade} from 'svelte/transition'

    function toUTF8Array(str) {
        var utf8 = [];
        for (var i = 0; i < str.length; i++) {
            var charcode = str.charCodeAt(i);
            if (charcode < 0x80) utf8.push(charcode);
            else if (charcode < 0x800) {
                utf8.push(0xc0 | (charcode >> 6),
                        0x80 | (charcode & 0x3f));
            } else if (charcode < 0xd800 || charcode >= 0xe000) {
                utf8.push(0xe0 | (charcode >> 12),
                        0x80 | ((charcode >> 6) & 0x3f),
                        0x80 | (charcode & 0x3f));
            }
            // surrogate pair
            else {
                i++;
                // UTF-16 encodes 0x10000-0x10FFFF by
                // subtracting 0x10000 and splitting the
                // 20 bits of 0x0-0xFFFFF into two halves
                charcode = 0x10000 + (((charcode & 0x3ff) << 10)
                        | (str.charCodeAt(i) & 0x3ff));
                utf8.push(0xf0 | (charcode >> 18),
                        0x80 | ((charcode >> 12) & 0x3f),
                        0x80 | ((charcode >> 6) & 0x3f),
                        0x80 | (charcode & 0x3f));
            }
        }
        return utf8;
    }

    function fromUTF8Array(data) { // array of bytes
        var str = '',
                i;

        for (i = 0; i < data.length; i++) {
            var value = data[i];

            if (value < 0x80) {
                str += String.fromCharCode(value);
            } else if (value > 0xBF && value < 0xE0) {
                str += String.fromCharCode((value & 0x1F) << 6 | data[i + 1] & 0x3F);
                i += 1;
            } else if (value > 0xDF && value < 0xF0) {
                str += String.fromCharCode((value & 0x0F) << 12 | (data[i + 1] & 0x3F) << 6 | data[i + 2] & 0x3F);
                i += 2;
            } else {
                // surrogate pair
                var charCode = ((value & 0x07) << 18 | (data[i + 1] & 0x3F) << 12 | (data[i + 2] & 0x3F) << 6 | data[i + 3] & 0x3F) - 0x010000;

                str += String.fromCharCode(charCode >> 10 | 0xD800, charCode & 0x03FF | 0xDC00);
                i += 3;
            }
        }

        return str;
    }

    let selected;
    let focused;

    let keyStr = '';
    let keyBytes = '';

    let firstForm = '';
    let secondForm = '';

    let notificationText = '';
    let notificationIsVisible = false;

    let purpose = 'encrypt';
    $: purpose = (selected === 'Шифровка') ? 'encrypt' : 'decrypt';

    const strFocused = () => focused = 'str';
    const byFocused = () => focused = 'by';

    $: if (focused === 'str') {
        let bytes = toUTF8Array(keyStr || '').map(v => `0x${Number(v).toString(16)}`)
        if (bytes.length > 8) {
            bytes.splice(8)
            keyStr = fromUTF8Array(bytes)
        }
        keyBytes = bytes.toString()
    } else if (focused === 'by') {
        let bytes = (keyBytes || '').split(',').map(v => parseInt((v || '').substr(2), 16))
        if (bytes.length > 8) {
            bytes.splice(8)
            keyBytes = bytes.map(v => `0x${Number(v).toString(16)}`).toString()
        }
        keyStr = keyBytes.length === 0 ? '' : fromUTF8Array(bytes)
    }

    async function fetchData() {
        return fetch(`https://fast-retreat-25276.herokuapp.com/${purpose}?key=${keyStr}`, {
            method: 'POST',
            headers: {
                'Content-Type': 'text/plain;charset=UTF-8'
            },
            body: selected === 'Шифровка' ? firstForm : secondForm
        }).then(v => {
            if (!v.ok) {
                throw new Error(`Code: ${v.status}, message: ${v.statusText}`);
            }

            return v;
        }).then(v => {
            return v.text();
        }).catch(error => {
            notificationText = error;
            notificationIsVisible = true;
        });
    }

    function formatByteSuffix(b) {
        if (b === 1) return 'байт';

        if (b >= 2 && b <= 4) return 'байта';

        return 'байтов';
    }

    async function handleClick() {
        let bytesInKey = toUTF8Array(keyStr || '').length;
        if (bytesInKey < 8) {
            notificationText = `Ключ слишком короткий: ${bytesInKey} ${formatByteSuffix(bytesInKey)}, необходимо 8 байтов`;
            notificationIsVisible = true;
            return
        }

        const data = await fetchData();

        if (selected === 'Шифровка') {
            secondForm = data;
        } else {
            firstForm = data;
        }
    }

    function hideNotification() {
        notificationIsVisible = false;
    }
</script>
{#if notificationIsVisible === true}
    <div transition:fade class="notification is-danger">
        <button class="delete" on:click={hideNotification}></button>
        {notificationText}
    </div>
{/if}
<section class="section">
    <div class="container has-text-centered">
        <div class="field">
            <div class="control">
                <label>
                    <input class="input" on:focus={strFocused} bind:value={keyStr} placeholder="Ключ шифрования">
                </label>
            </div>
        </div>
        <div class="field">
            <div class="control">
                <label>
                    <input class="input" on:focus={byFocused} bind:value={keyBytes}
                           placeholder="Ключ шифрования (байты)">
                </label>
            </div>
        </div>
    </div>
</section>
<section class="section">
    <div class="container">
        <div class="columns">
            <!-- зачем нужны компоненты лол -->
            {#if selected === 'Шифровка'}
                <div class="column">
                    <label>
                    <textarea class="textarea is-info" bind:value={firstForm} placeholder="Обычный текст"
                              rows="15"></textarea>
                    </label>
                </div>
                <div class="column">
                    <label>
                    <textarea class="textarea is-danger" bind:value={secondForm} placeholder="Шифровка"
                              rows="15" readonly></textarea>
                    </label>
                </div>
            {:else}
                <div class="column">
                    <label>
                    <textarea class="textarea is-danger" bind:value={secondForm} placeholder="Шифровка"
                              rows="15"></textarea>
                    </label>
                </div>
                <div class="column">
                    <label>
                    <textarea class="textarea is-info" bind:value={firstForm} placeholder="Обычный текст"
                              rows="15" readonly></textarea>
                    </label>
                </div>
            {/if}
        </div>
    </div>
</section>
<section class="section">
    <div class="container has-text-centered">
        <div class="select is-primary">
            <label>
                <select bind:value={selected}>
                    <option>Шифровка</option>
                    <option>Расшифровка</option>
                </select>
            </label>
        </div>

        <button class="button is-primary" on:click={handleClick}>
            <!-- И мне не стыдно! -->
            {#if selected === 'Шифровка'}
                Зашифровать
            {:else}
                Расшифровать
            {/if}
        </button>
    </div>
</section>
<footer class="footer">
    <div class="content has-text-centered">
        <p>
            Лабораторная работа №4 <b>"Стандарты симметричного шифрования"</b> по дисциплине <b>"Организация и
            технология защиты информации"</b>
        </p>
        <p>
            Сделали студенты группы ПИ-72 <a href="https://github.com/andiogenes">Антон Завьялов</a> и <a
                href="https://github.com/NikolayCheremnov">Николай Черемнов</a>
        </p>
    </div>
</footer>