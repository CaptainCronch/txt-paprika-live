<script lang="js">
    import * as json from "$lib/assets/dialogue.json"
    import { noise } from "$lib/assets/noise.js"
    import { onMount } from "svelte";

    const defaultCharSpeed = 30
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

    let charSpeed = defaultCharSpeed
    let totalDialogue = "" // total text in current dialogue bubble
    /** @type {{ text: string; color: string; moveEffect: string;}[]} */
    let splitDialogue = [] // array of all section objects in current bubble
    let sectionIndex = 0 // index of section we are currently writing to visible from
    let bubbleIndex = 0
    let isSectionNew = true
    let invisible = ""
    let visible = ""
    let texting = false
    let bubbleComplete = false
    let timeoutID = 0

    loadDialogue(bubbleIndex)

    /** @param {number} num */
    function loadDialogue(num){ // load next bubble
        // content has all text from conversation. content[0] is first dialogue block. 
        // content[x][0] is first section from x dialogue block (separated so that different effects may be applied)
        if (num + 1 > json.content.length || num < 0) {return false}
        bubbleComplete = false
        texting = false
        visible = ""
        sectionIndex = 0
        totalDialogue = ""
        json.content[num].forEach(element => {totalDialogue += element.text});
        invisible = totalDialogue
        splitDialogue = structuredClone(json.content[num])
    }

    function dialogueClick(dir = 1) {
        // console.log("clicked dialogue. texting is " + texting + ". bubbleComplete is " + bubbleComplete + ". first text is " + json.content[bubbleIndex][0].text)
        if (bubbleComplete === true) {
            bubbleIndex += dir // load next bubble if all text has been written
            if (loadDialogue(bubbleIndex) == false) {
                bubbleIndex -= dir
                return
            }
            // console.log("next bubble")
        } else if (texting === true) {
            clearTimeout(timeoutID) // skip dialogue if in the middle of writing
            addLetter(true)
            // console.log("skipped writing")
            return
        }
        // console.log("starting dialogue")
        texting = true
        addLetter()
    }

    function nextClick() {
        dialogueClick(1)
    }

    function backClick() {
        dialogueClick(-1)
    }

    /** @param {KeyboardEvent} event */
    function dialogueButton(event) {
        if (event.key == "ArrowRight" || event.key == "d" || event.key == " ") {nextClick()}
        else if (event.key == "ArrowLeft" || event.key == "a") {backClick()}
    }

    function addLetter(skip = false) {
        if (texting == false) {return}
        // if (invisible.length <= 0) {texting = false; return}
        if (splitDialogue[sectionIndex].text.length <= 0) {
            if (sectionIndex < splitDialogue.length - 1) { // if there's another section of text move to it when the current section runs out
                sectionIndex += 1
                isSectionNew = true
            } else {
                console.log("finished writing")
                bubbleComplete = true
                texting = false
                return
            }
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

        if (skip) {addLetter(true)} // if skip is on instantly write next letter
        else {timeoutID = setTimeout(addLetter, charSpeed)}
    }

    /**
     * @param {number} num
     * @param {number} lower
     * @param {number} upper
     */
    function clamp(num, lower, upper) {
        return Math.min(Math.max(num, lower), upper);
    }

    onMount(() => { // text effects
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
    <div class="dialogue-background-layer-back">
        <div class="dialogue-background-layer-front">
            <p class="dialogue-text"><span class="visible-text">{@html visible}</span>{@html invisible}</p>
            <div class="controls">
                <button title="Dialogue Back" class="caret caret-back" onclick={backClick}>
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 0.48 0.48" transform="matrix(6.123233995736766e-17,1,1,-6.123233995736766e-17,0,0)">
                    <path d="M0.405 0.35H0.07l0.1685 -0.25Z"/>
                </svg>
                </button>
                <button title="Dialogue Next" class="caret caret-next" onclick={nextClick}>
                    <svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 0.48 0.48" transform="matrix(1,0,0,-1,0,0)">
                    <path d="M0.405 0.35H0.07l0.1685 -0.25Z"/>
                </svg>
                </button>
            </div>
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
    overflow-wrap: break-word;
}

.visible-text {
    color: white;
}

.controls {
    display: flex;
    justify-content: end;
}

.caret {
    display: block;
    width: 2em;
    height: 2em;
    margin: 0;
    border-radius: 0;
    border: none;
    background-color: transparent;
    color: white;
    transform: translate(0px, 0px);
    transition: transform 50ms ease-out;
}

.caret svg {
    width: 100%;
    height: 100%;
}

.caret-back:active {
    transform: translate(-5px, 0px);
}

.caret-next:active {
    transform: translate(0px, 5px);
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