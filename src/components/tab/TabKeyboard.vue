
<!--
****************************************************************************
#                                                                            #
#    blikvm                                                                  #
#                                                                            #
#    Copyright (C) 2021-present     blicube <info@blicube.com>               #
#                                                                            #
#    This program is free software: you can redistribute it and/or modify    #
#    it under the terms of the GNU General Public License as published by    #
#    the Free Software Foundation, either version 3 of the License, or       #
#    (at your option) any later version.                                     #
#                                                                            #
#    This program is distributed in the hope that it will be useful,         #
#    but WITHOUT ANY WARRANTY; without even the implied warranty of          #
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the           #
#    GNU General Public License for more details.                            #
#                                                                            #
#    You should have received a copy of the GNU General Public License       #
#    along with this program.  If not, see <https://www.gnu.org/licenses/>.  #
#                                                                            #
****************************************************************************
-->
<template>
  <div class="keyboardContainer" ref="kb" @mousedown="onMouseDown">
    <div class="simple-keyboard-main"></div>

    <div class="controlArrows">
      <div class="simple-keyboard-control"></div>
      <div class="simple-keyboard-arrows"></div>
    </div>

    <div class="numPad">
      <div class="simple-keyboard-numpad"></div>
      <div class="simple-keyboard-numpadEnd"></div>
    </div>
  </div>
</template>

<script setup>
import Keyboard from "simple-keyboard";
import "simple-keyboard/build/css/index.css";
import { ref, onMounted,onUnmounted, watch } from 'vue';

const emit = defineEmits(['onChange', 'onKeyPress', 'onKeyReleased']);
const props = defineProps(['input']);
let keyboard = null;
let keyboardControlPad = null;
let keyboardArrows = null;
let keyboardNumPad = null;
let keyboardNumPadEnd = null;

const kb = ref(null);
const isDragging = ref(false);
const initialMouseX = ref(0);
const initialMouseY = ref(0);
const initialKeyboardX = ref(0);
const initialKeyboardY = ref(0);

const onChange = (input) => {
    // emit("onChange", input);
    // console.log("onChange Value", input);
};

const onKeyReleased = (input) => {
    emit("onKeyReleased", input);
    // console.log("onChange Value", input);
};

const onKeyPress = (button) => {
    emit("onKeyPress", button);
    // console.log("onKeyPress Value", button);
    if (
        button === "{shift}" ||
        button === "{shiftleft}" ||
        button === "{shiftright}" ||
        button === "{capslock}"
    )
    handleShift();
};

const handleShift = () => {
    let currentLayout = keyboard.options.layoutName;
    let shiftToggle = currentLayout === "default" ? "shift" : "default";
    console.log("layout:",shiftToggle);
    keyboard.setOptions({
        layoutName: shiftToggle
    });
};

const onMouseDown = (event) => {
  isDragging.value = true;
  initialMouseX.value = event.clientX;
  initialMouseY.value = event.clientY;
  initialKeyboardX.value = kb.value.offsetLeft;
  initialKeyboardY.value = kb.value.offsetTop;
  kb.value.style.cursor = 'move';
  window.addEventListener('mousemove', onMouseMove);
  window.addEventListener('mouseup', onMouseUp);
};

const onMouseUp = () => {
  isDragging.value = false;
  kb.value.style.cursor = 'default';
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('mouseup', onMouseUp);
};

const onMouseMove = (event) => {
  if (!isDragging.value) return;
  const deltaX = event.clientX - initialMouseX.value;
  const deltaY = event.clientY - initialMouseY.value;
  const parent = kb.value.parentElement;

  let newLeft = initialKeyboardX.value + deltaX;
  let newTop = initialKeyboardY.value + deltaY;

  newLeft = Math.max(0, Math.min(newLeft, parent.clientWidth - kb.value.clientWidth));
  newTop = Math.max(0, Math.min(newTop, parent.clientHeight - kb.value.clientHeight));

  kb.value.style.left = `${newLeft}px`;
  kb.value.style.top = `${newTop}px`;
};

onMounted(() => {
  let commonKeyboardOptions = {
    onChange: input => onChange(input),
    onKeyPress: button => onKeyPress(button),
    onKeyReleased: button => onKeyReleased(button),
    theme: "simple-keyboard hg-theme-default hg-layout-default",
    physicalKeyboardHighlight: true,
    syncInstanceInputs: true,
    mergeDisplay: true,
    debug: false
  };
  keyboard = new Keyboard(".simple-keyboard-main", {
    ...commonKeyboardOptions,
    layout: {
      default: [
        "{escape} {f1} {f2} {f3} {f4} {f5} {f6} {f7} {f8} {f9} {f10} {f11} {f12}",
        "` 1 2 3 4 5 6 7 8 9 0 - = {backspace}",
        "{tab} q w e r t y u i o p [ ] \\",
        "{capslock} a s d f g h j k l ; ' {enter}",
        "{shiftleft} z x c v b n m , . / {shiftright}",
        "{controlleft} {altleft} {metaleft} {space} {metaright} {altright}"
      ],
      shift: [
        "{escape} {f1} {f2} {f3} {f4} {f5} {f6} {f7} {f8} {f9} {f10} {f11} {f12}",
        "~ ! @ # $ % ^ & * ( ) _ + {backspace}",
        "{tab} Q W E R T Y U I O P { } |",
        '{capslock} A S D F G H J K L : " {enter}',
        "{shiftleft} Z X C V B N M < > ? {shiftright}",
        "{controlleft} {altleft} {metaleft} {space} {metaright} {altright}"
      ]
    },
    display: {
      "{escape}": "esc ⎋",
      "{tab}": "tab ⇥",
      "{backspace}": "backspace ⌫",
      "{enter}": "enter ↵",
      "{capslock}": "caps lock ⇪",
      "{shiftleft}": "shift ⇧",
      "{shiftright}": "shift ⇧",
      "{controlleft}": "ctrl ⌃",
      "{controlright}": "ctrl ⌃",
      "{altleft}": "alt ⌥",
      "{altright}": "alt ⌥",
      "{metaleft}": "cmd ⌘",
      "{metaright}": "cmd ⌘"
    }
  });
  keyboardControlPad = new Keyboard(".simple-keyboard-control", {
    ...commonKeyboardOptions,
    layout: {
      default: [
        "{prtscr} {scrolllock} {pause}",
        "{insert} {home} {pageup}",
        "{delete} {end} {pagedown}"
      ]
    }
  });
  keyboardArrows = new Keyboard(".simple-keyboard-arrows", {
    ...commonKeyboardOptions,
    layout: {
      default: ["{arrowup}", "{arrowleft} {arrowdown} {arrowright}"]
    }
  });
  keyboardNumPad = new Keyboard(".simple-keyboard-numpad", {
    ...commonKeyboardOptions,
    layout: {
      default: [
        "{numlock} {numpaddivide} {numpadmultiply}",
        "{numpad7} {numpad8} {numpad9}",
        "{numpad4} {numpad5} {numpad6}",
        "{numpad1} {numpad2} {numpad3}",
        "{numpad0} {numpaddecimal}"
      ]
    }
  });
  keyboardNumPadEnd = new Keyboard(".simple-keyboard-numpadEnd", {
    ...commonKeyboardOptions,
    layout: {
      default: ["{numpadsubtract}", "{numpadadd}", "{numpadenter}"]
    }
  });

  const parent = kb.value.parentElement;
  const parentWidth = parent.clientWidth;
  const keyboardWidth = kb.value.clientWidth;

  kb.value.style.left = `${(parentWidth - keyboardWidth) / 2}px`;
  kb.value.style.top = `${parent.clientHeight - kb.value.clientHeight}px`;

});

onUnmounted(() => {
  window.removeEventListener('mousemove', onMouseMove);
  window.removeEventListener('mouseup', onMouseUp);

  if (keyboard) {
    keyboard.destroy();
    keyboard = null;
  }
  if (keyboardControlPad) {
    keyboardControlPad.destroy();
    keyboardControlPad = null;
  }
  if (keyboardArrows) {
    keyboardArrows.destroy();
    keyboardArrows = null;
  }
  if (keyboardNumPad) {
    keyboardNumPad.destroy();
    keyboardNumPad = null;
  }
  if (keyboardNumPadEnd) {
    keyboardNumPadEnd.destroy();
    keyboardNumPadEnd = null;
  }
});


watch(() => props.input, (newValue, oldValue) => {
  console.log("input:",newValue);
  // keyboard.setInput(newValue);
});

</script>

<style scoped>

.keyboardContainer {
  display: flex;
  background-color: rgba(0, 0, 0, 0.637);
  justify-content: center;
  width: 1024px;
  margin: 0 auto;
  border-radius: 5px;
  position:absolute;
}

.simple-keyboard.hg-theme-default {
  display: inline-block;
}

.simple-keyboard-main.simple-keyboard {
  width: 640px;
  min-width: 640px;
  background-color: transparent;
}

.simple-keyboard-main.simple-keyboard .hg-row:first-child {
  margin-bottom: 10px;
}

.simple-keyboard-arrows.simple-keyboard {
  align-self: flex-end;
  background-color: transparent;
}

.simple-keyboard .hg-button.selectedButton {
  background-color: rgba(5, 25, 70, 0.53);
  color: white;
}

.simple-keyboard .hg-button.emptySpace {
  pointer-events: none;
  background-color: transparent;
  border: none;
  box-shadow: none;
}

.simple-keyboard-arrows .hg-row {
  justify-content: center;
}

.simple-keyboard-arrows .hg-button {
  width: 50px;
  flex-grow: 0;
  justify-content: center;
  display: flex;
  align-items: center;
}

.controlArrows {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-flow: column;
}

.simple-keyboard-control.simple-keyboard {
  background-color: transparent;
}

.simple-keyboard-control.simple-keyboard .hg-row:first-child {
  margin-bottom: 10px;
}

.simple-keyboard-control .hg-button {
  width: 50px;
  flex-grow: 0;
  justify-content: center;
  display: flex;
  align-items: center;
}

.numPad {
  display: flex;
  align-items: flex-end;
}

.simple-keyboard-numpad.simple-keyboard {
  background-color: transparent;
}

.simple-keyboard-numpad.simple-keyboard {
  width: 160px;
}

.simple-keyboard-numpad.simple-keyboard .hg-button {
  width: 50px;
  justify-content: center;
  display: flex;
  align-items: center;
}

.simple-keyboard-numpadEnd.simple-keyboard {
  width: 50px;
  background: transparent;
  margin: 0;
  padding: 5px 5px 5px 0;
}

.simple-keyboard-numpadEnd.simple-keyboard .hg-button {
  align-items: center;
  justify-content: center;
  display: flex;
}

.simple-keyboard-numpadEnd .hg-button.hg-standardBtn.hg-button-plus {
  height: 85px;
}

.simple-keyboard-numpadEnd.simple-keyboard .hg-button.hg-button-enter {
  height: 85px;
}

.simple-keyboard.hg-theme-default .hg-button.hg-selectedButton {
  background: rgba(5, 25, 70, 0.53);
  color: white;
}

.hg-button.hg-functionBtn.hg-button-space {
  width: 350px;
}

.active-toolbar-btn {
    color: rgb(52, 231, 8);
    border: 2px solid rgb(52, 231, 8);
    border-radius: 50%;
    padding: 5px;
    margin-right: 15px;
    transition: color 0.3s ease;
}

.inactive-toolbar-btn {
    color:red;
    border: 2px solid red;
    border-radius: 50%;
    padding: 5px;
    margin-right: 15px;
    transition: color 0.3s ease;
}
</style>