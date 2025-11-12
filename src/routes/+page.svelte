<script lang="js">
    import * as json from "$lib/assets/dialogue.json"
    import { noise } from "$lib/assets/noise.js"
    import { onMount } from "svelte";

    const punctuationSpeeds = {
        " ": 20,
        ",": 200,
        ".": 300,
        "!": 250,
        "?": 250,
        ":": 230,
        ";": 230,
        "/": 150,
    }

    let defaultCharSpeed = 30
    let totalDialogue = "" // total text in current dialogue bubble
    /** @type {{ text: string; color: string; moveEffect: string;}[]} */
    let splitDialogue = []
    let sectionIndex = 0
    let isSectionNew = true
    let invisible = ""
    let visible = ""
    let texting = false

    // content has all text from conversation. content[0] is first dialogue block. content[x][0] is first section from x dialogue block (separated so that different effects may be applied)
    json.content[0].forEach(element => {totalDialogue += element.text});
    invisible = totalDialogue
    splitDialogue = json.content[0]

    function dialogueClick() {
        if (texting === true) {
            texting = false
            visible = totalDialogue
            invisible = ""
            return
        }
        texting = true
        setTimeout(addLetter, 10)
    }

    /** @param {KeyboardEvent} event */
    function dialogueButton(event) {
        if (event.key != " ") {return}
        dialogueClick()
    }

    function addLetter() {
        if (texting == false) {return}
        if (invisible.length <= 0) {texting = false; return}
        if (splitDialogue[sectionIndex].text.length <= 0) {
            if (sectionIndex < splitDialogue.length - 1) { // if there's another section of text move to it when the current section runs out
                sectionIndex += 1
                isSectionNew = true
            } else {return}
        }

        if (isSectionNew) { // add color span
            if (splitDialogue[sectionIndex].color != "") {visible += `<span style="color: ${splitDialogue[sectionIndex].color};">`}
        }

        if (splitDialogue[sectionIndex].text.charAt(0) == "\n") { // make sure newlines actually newline in html
            visible += "<br>"
        } else if (splitDialogue[sectionIndex].moveEffect == "shake" && splitDialogue[sectionIndex].text.charAt(0) != " ") {
            visible += `<span class="shake" style="position:relative;top:0px;left:0px;">${splitDialogue[sectionIndex].text.charAt(0)}</span>`
        } else if (splitDialogue[sectionIndex].moveEffect == "float" && splitDialogue[sectionIndex].text.charAt(0) != " ") {
            visible += `<span class="float">${splitDialogue[sectionIndex].text.charAt(0)}</span>`
        } else if (splitDialogue[sectionIndex].moveEffect == "sweep" && splitDialogue[sectionIndex].text.charAt(0) != " ") {
            visible += `<span class="sweep">${splitDialogue[sectionIndex].text.charAt(0)}</span>`
        } else if (splitDialogue[sectionIndex].moveEffect == "wave" && splitDialogue[sectionIndex].text.charAt(0) != " ") {
            visible += `<span class="wave">${splitDialogue[sectionIndex].text.charAt(0)}</span>`
        } else {
            visible += splitDialogue[sectionIndex].text.charAt(0)
        }
        

        let charSpeed = defaultCharSpeed 
        let punctuation = Object.keys(punctuationSpeeds)
        for (const value of punctuation.keys()) { // increase next character delay if current chraracter is punctuation
            if (splitDialogue[sectionIndex].text.charAt(0) == punctuation[value]) {
                charSpeed = Object.values(punctuationSpeeds)[value]
                break
            }
        }

        isSectionNew = false
        splitDialogue[sectionIndex].text = splitDialogue[sectionIndex].text.slice(1)
        invisible = invisible.slice(1) // delete first character

        if (splitDialogue[sectionIndex].text.length <= 0) {
            if (splitDialogue[sectionIndex].color != "") {visible += `</span>`}
        }
        setTimeout(addLetter, charSpeed)
    }

    /**
     * @param {number} num
     * @param {number} lower
     * @param {number} upper
     */
    function clamp(num, lower, upper) {
        return Math.min(Math.max(num, lower), upper);
    }

    onMount(() => {
        let shakeLastTime = document.timeline.currentTime
        /** @param {number} time */
        function shake(time) {
            // const maxRange = 4
            const power = 2
            const timeFactor = 100
            // let delta = (time - Number(shakeLastTime))/1000
            let shakers = document.getElementsByClassName("shake")
            let i = 0
            for (let element of shakers) {
                noise().seed(Math.random())
                element.setAttribute("style",
                    `position:relative;
                    top:${(((noise().simplex2(time/timeFactor + i, time/timeFactor + i) * power) - (power / 2)))}px;
                    left:${(((noise().perlin2(time/timeFactor + i, time/timeFactor + i) * power) - (power / 2)))}px;`
                ) // per-character shake effect
                i += 1
            }
            // shakeLastTime = time
            requestAnimationFrame(shake)
        }
        requestAnimationFrame(shake)

        /** @param {number} time */
        function float(time) {
            const speed = 2 / 1000
            const frequency = 0.2
            const amplitude = 8
            let floaters = document.getElementsByClassName("float")
            let i = 0
            for (let element of floaters) {
                element.setAttribute("style", `position:relative;top:${Math.sin((time * speed) - i) * amplitude}px;`) // per-character float effect
                i += frequency
            }
            requestAnimationFrame(float)
        }
        requestAnimationFrame(float)

        /** @param {number} time */
        function sweep(time) {
            const speed = 2 / 1000
            const frequency = 0.2
            const amplitude = 8
            let sweepers = document.getElementsByClassName("sweep")
            let i = 0
            for (let element of sweepers) {
                element.setAttribute("style", `position:relative;left:${Math.cos((time * speed) - i) * amplitude}px;`) // per-character sweep effect
                i += frequency
            }
            requestAnimationFrame(sweep)
        }
        requestAnimationFrame(sweep)

        /** @param {number} time */
        function wave(time) {
            const speed = 2 / 1000
            const frequency = 0.2
            const amplitude = 8
            let wavers = document.getElementsByClassName("wave")
            let i = 0
            for (let element of wavers) {
                element.setAttribute("style", `position:relative;top:${Math.sin((time * speed) - i) * amplitude}px;left:${Math.cos((time * speed) - i) * amplitude}px;`) // per-character wave effect
                i += frequency
            }
            requestAnimationFrame(wave)
        }
        requestAnimationFrame(wave)
    })
</script>

<svelte:window onkeyup={dialogueButton}></svelte:window>
<div class="dialogue-box">
    <!-- svelte-ignore a11y_no_static_element_interactions -->
    <!-- svelte-ignore a11y_click_events_have_key_events -->
    <div class="dialogue-background-layer-back">
        <div class="dialogue-background-layer-front">
            <p class="dialogue-text"><span class="visible-text">{@html visible}</span>{@html invisible}</p>
            <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" class="caret-down" viewBox="0 0 16 16" onclick={dialogueClick}>
                <path d="M7.247 11.14 2.451 5.658C1.885 5.013 2.345 4 3.204 4h9.592a1 1 0 0 1 .753 1.659l-4.796 5.48a1 1 0 0 1-1.506 0z"/>
            </svg>
        </div>
    </div>
</div>

<style>
@import "$lib/assets/style.css";

.dialogue-box {
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
    /* border: 5px black solid; */
    /* text-overflow: ellipsis; */
    overflow: visible;
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

.dialogue-text {
    height: 6em;
    color: transparent;
}

.visible-text {
    color: white;
}

.caret-down {
    display: block;
    margin-left: auto;
    width: 2em;
    height: 2em;
}

/* .shake {
    animation: 0.2s linear infinite shake;
    animation-delay: calc(var(--rand) * 100ms);
    position: relative;
    top: 0;
    left: 0;
} */

/* @keyframes shake {
    0% {
        top: 2px;
        left: 2px;
    }
    25% {
        top: -2px;
        left: -2px;
    }
    50% {
        top: -2px;
        left: 2px;
    }
    75% {
        top: 2px;
        left: -2px;
    }
    100% {
        top: 2px;
        left: 2px;
    }
} */

/* .float {
    animation: 1s linear alternate infinite float;
    position: relative;
}

@keyframes float {
    from {
        top: 2px;
    }
    to {
        top: -2px;
    }
} */
</style>