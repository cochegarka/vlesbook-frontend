<script>
    import 'bulma/css/bulma.css'
    import '@fortawesome/fontawesome-free/css/all.css'

    let selected;

    let keyStr = '';
    let keyBytes;
</script>

<section class="section">
    <div class="container has-text-centered">
        <div class="field">
            <div class="control">
                <label>
                    <input class="input" bind:value={keyStr} placeholder="Ключ шифрования">
                </label>
            </div>
        </div>
        <div class="field">
            <div class="control">
                <label>
                    <input class="input" bind:value={keyBytes} placeholder="Ключ шифрования (байты)">
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