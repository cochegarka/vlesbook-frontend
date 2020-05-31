<script>
    import 'bulma/css/bulma.css'
    import '@fortawesome/fontawesome-free/css/all.css'

    function toUTF8Array(str) {
        var utf8 = [];
        for (var i=0; i < str.length; i++) {
            var charcode = str.charCodeAt(i);
            if (charcode < 0x80) utf8.push(charcode);
            else if (charcode < 0x800) {
                utf8.push(0xc0 | (charcode >> 6),
                        0x80 | (charcode & 0x3f));
            }
            else if (charcode < 0xd800 || charcode >= 0xe000) {
                utf8.push(0xe0 | (charcode >> 12),
                        0x80 | ((charcode>>6) & 0x3f),
                        0x80 | (charcode & 0x3f));
            }
            // surrogate pair
            else {
                i++;
                // UTF-16 encodes 0x10000-0x10FFFF by
                // subtracting 0x10000 and splitting the
                // 20 bits of 0x0-0xFFFFF into two halves
                charcode = 0x10000 + (((charcode & 0x3ff)<<10)
                        | (str.charCodeAt(i) & 0x3ff));
                utf8.push(0xf0 | (charcode >>18),
                        0x80 | ((charcode>>12) & 0x3f),
                        0x80 | ((charcode>>6) & 0x3f),
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
    let keyBytes;

    const strFocused = () => focused = 'str';
    const byFocused = () => focused = 'by';

    $: if (focused === 'str') {
        keyBytes = toUTF8Array(keyStr || '').map(v => `0x${Number(v).toString(16)}`).toString()
    } else if (focused === 'by') {
        let bytes = (keyBytes || '').split(',').map(v => parseInt((v || '').substr(2), 16))
        keyStr = fromUTF8Array(bytes)
    }
</script>

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
            <div class="column">
                <label>
                    <textarea class="textarea is-info" placeholder="Обычный текст" rows="15"></textarea>
                </label>
            </div>
            <div class="column">
                <label>
                    <textarea class="textarea is-success" placeholder="Шифровка" rows="15"></textarea>
                </label>
            </div>
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

        <button class="button is-primary">
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
            <strong>Передаю привет семье: маме, папе, брату, бабушке, кошке Анфисе.</strong>
        </p>
    </div>
</footer>