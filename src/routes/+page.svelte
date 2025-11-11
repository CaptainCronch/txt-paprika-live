<script lang="js">
    let lorem = "Lorem ipsum dolor sit amet consectetur adipisicing elit. Eveniet quod qui nisi reiciendis numquam quas, corporis repellendus repellat molestiae! Placeat dolorum tempore autem nam possimus ipsum maiores consequatur! Asperiores, obcaecati."
    let invisible = ""
    let visible = ""
    let texting = false

    function addText(){
        if (texting === true) {return}
        texting = true

        // move letters from the beginning of invisible to the end of invisible, with a time delay. change the delay based on which character it is, so that punctuation can pause.
    }
</script>

<div id="dialogue-box">
    <div class="dialogue-background-layer-back" onclick={addText}>
        <div class="dialogue-background-layer-front">
            <p id="dialogue-text"><span class="visible-text">{visible}</span>{invisible}</p>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="caret-down" viewBox="0 0 16 16">
                <path d="M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z"/>
            </svg>
        </div>
    </div>
</div>

<style>
@import "$lib/assets/style.css";

#dialogue-box {
    color: white;
    font-size: 24px;
    width: 40em;
    height: 10em;
    position: absolute;
    margin: 0 auto;
    bottom: 2em;
    left: 50%;
    transform: translateX(-50%);
    /* border-radius: 2em; */
    border: black solid;
    text-overflow: ellipsis;
    overflow: hidden;
    background-color: black;
}

.dialogue-background-layer-back {
    background-image: url("$lib/assets/64-checkers.png");
    animation: 6s linear infinite back-slide;
}

@keyframes back-slide {
    from {background-position: 0 0;}
    to {background-position: 128px 128px;}
}

.dialogue-background-layer-front {
    background-image: url("$lib/assets/128-checkers.png");
    padding: 1em 2em;
    animation: 3s linear infinite front-slide;
}

@keyframes front-slide {
    from {background-position: 0 0;}
    to {background-position: -128px 128px;}
}

#dialogue-text {
    height: 6em;
    text-overflow: ellipsis;
    overflow: hidden;
}

.caret-down {
    display: block;
    margin-left: auto;
    width: 2em;
    height: 2em;
}
</style>