<template>
  <div :class="setClass" class="vue-formulize-grid">
    <input autocomplete="off" id="formula" @input="updateFormula" v-model="formulize" />
    <div
      :class="{'active': isActive,  'error-bracket': errorName == 'bracket', 'error-measurement': errorName == 'measurement'}"
      @click.stop="addEvent()"
      v-html="formulaHtml"
      ref="vueFormulize"
      id="vueFormulize"
      class="vue-formulize"
    ></div>
    <!-- start status  -->
    <p
      v-if="status"
      :type="status ? status.type : null"
      class="vue-formulize-error"
    >{{status.message}}</p>
    <!-- end status  -->
    {{output}}
  </div>
</template>


<script>
export default {
  name: "VueFormulize",
  props: {
    options: {
      type: Array,
      default: () => []
    },
    // formulize active/disable toggle
    isEditable: {
      type: Boolean,
      default: true
    },
    // string input to formulize
    input: {
      type: String,
      default: null
    },
    // custom class
    className: {
      type: String,
      default: ""
    },
    // valid keys
    keys: {
      type: Array,
      default: () => ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
    },
    // mathematical operators
    operators: {
      type: Array,
      default: () => ["*", "-", "/", "+"]
    },
    error: {
      type: Object,
      default: () => {}
    },
    customElement: {
      type: Array,
      default: () => []
    }
  },

  watch: {
    // if input value is changed
    output: function() {
      this.validation();
    }
  },

  computed: {
    setClass() {
      let className = "";
      if (this.className) className = this.className;
      if (this.status) className += " error";
      if (!className) return null;
      return className;
    }
  },

  data() {
    return {
      customClass: this.className,
      isActive: false,
      output: null,
      errorName: null,
      formulize: null,
      formulaHtml: "",
      capturedKey: null,
      cursorPosition: 0,
      specials: ["(", "{", "}", ")"],
      status: null,
      capturedText: ""
    };
  },

  methods: {
    /**
     * Target click
     * @param {Object} e
     */
    documentClick(e) {
      try {
        let el = this.$refs.vueFormulize;
        let target = e.target;
        if (el !== target && !el.contains(target)) {
          this.onBlur();
        }
      } catch (error) {
        // console.log(error)
      }
    },

    /**
     * Set error
     * @param {String} type, message
     */
    setStatus(type, message) {
      if (!type) {
        this.status = null;
      } else {
        this.status = { type, message };
      }
    },

    /**
     *  Events are added when clicked formula grid
     */
    addEvent() {
      if (!this.isActive && this.isEditable) {
        this.onFocus();
        document.addEventListener("keydown", this.captureKey);
        document.addEventListener("paste", this.setClipboard);
        this.createCursor();
        this.isActive = true;
      }
    },

    /**
     * Add html element to formula grid
     * @param {String} tagName, className,
     * @param {Number} tagName, className,
     */
    createHtmlElement(tagName, className, value, id, event) {
      let element = document.createElement(tagName);
      element.setAttribute("class", className);
      element.setAttribute("value", value);
      if (id) element.setAttribute("id", id);
      element.innerText = value;
      if (event) {
        element.addEventListener("onclick", function(e) {
          console.log(2);
        });
      }
      return element;
    },

    createCursor() {
      let element = document.querySelector("#vueFormulize");
      this.removeCursor();
      this.cursorPosition = element.childNodes.length;
      let htmlCode = this.createHtmlElement(
        "span",
        "formula-cursor",
        "",
        "formulaCursor"
      );
      this.addElementToFormula(htmlCode);
      this.cursorMove("right");
    },

    removeCursor() {
      try {
        var cursor = document.getElementById("formulaCursor");
        if (cursor) cursor.parentNode.removeChild(cursor);
      } catch (error) {
        // console.error(error)
      }
    },

    captureKey(e) {
      // console.log(this.capturedKey, e)
      this.capturedKey = e;
      this.onFocus();
      if (this.capturedKey.key == "ArrowLeft") this.cursorMove("left");
      if (this.capturedKey.key == "ArrowRight") this.cursorMove("right");
      if (this.capturedKey.key == "Backspace") this.removeItem();
      if (this.capturedKey.key == "Delete") this.deleteItem();
      this.captureCustomElement(e);
    },

    captureCustomElement(e) {
      this.capturedText += e.key;

      console.log(this.capturedText);
      for (let i = 0; i < this.customElement.length; i++) {
        const item = this.customElement[i];
        if (this.capturedText.search(item.key) > 0) {
          this.capturedText = "";
          this.addElementToFormula(
            this.createHtmlElement(
              item.element,
              item.class,
              item.value,
              `${item.element}${item.value}`,
              item.event
            )
          );
          this.cursorMove("right");
        }
      }
    },

    removeItem() {
      try {
        let element = document.querySelector("#vueFormulize");
        let index = this.cursorPosition - 1;
        if (element.childNodes[index].id === "formulaCursor")
          this.cursorMove("left");
        if (element.childNodes[index].id !== "formulaCursor")
          element.removeChild(element.childNodes[index]);
        this.cursorMove("left");
      } catch (error) {
        // console.log(error)
      }
    },

    deleteItem() {
      try {
        let element = document.querySelector("#vueFormulize");
        let index = this.cursorPosition + 1;
        if (element.childNodes[index].id === "formulaCursor")
          this.cursorMove("right");
        if (element.childNodes[index].id !== "formulaCursor")
          element.removeChild(element.childNodes[index]);
      } catch (error) {
        // console.log(error)
      }
    },

    /**
     * Navigation cursor
     * if you are set a new position of cursor,
     * use index value
     * @param {String} direction,
     * @param {Number} index
     */
    cursorMove(direction, index) {
      try {
        let element = document.querySelector("#vueFormulize");
        let cursor = document.querySelector("#formulaCursor");
        let child = [...element.childNodes];
        if (this.cursorPosition < child.length - 1 && direction == "right") {
          this.cursorPosition++;
          element.insertBefore(cursor, child[this.cursorPosition + 1]);
        }
        if (this.cursorPosition > 0 && direction == "left") {
          this.cursorPosition--;
          element.insertBefore(cursor, child[this.cursorPosition]);
        }
        if (!direction && index) this.cursorPosition = index;
      } catch (error) {
        return error;
      }
    },

    updateFormula() {
      let element = document.querySelector("#vueFormulize");
      let child = element.childNodes;
      let htmlCode = null;
      let order = child.length - 1;

      // move cursor
      if (order == this.cursorPosition) order++;

      if (this.capturedKey.key == "(")
        htmlCode = this.createHtmlElement("span", "item secondary", "(");

      if (this.capturedKey.key == ")")
        htmlCode = this.createHtmlElement("span", "item secondary", ")");

      for (const o of this.operators) {
        if (this.capturedKey.key == o)
          htmlCode = this.createHtmlElement("span", "item operator space-m", o);
      }

      for (const n of this.keys) {
        if (this.capturedKey.key == n)
          htmlCode = this.createHtmlElement("span", "item", n);
      }

      if (this.capturedKey.code == "Space") htmlCode = null;

      if (htmlCode) {
        this.addElementToFormula(htmlCode);
        this.cursorMove("right");
      }

      this.getFormulaString();
    },

    addElementToFormula(item) {
      try {
        let element = document.querySelector("#vueFormulize");
        let child = element.childNodes;
        element.insertBefore(item, child[this.cursorPosition]);
      } catch (error) {
        // console.log(error)
      }
    },

    addMeasurement(defaultSelectTitle) {
      let element = document.querySelector("#vueFormulize");
      let child = element.childNodes;
      let order = child.length - 1;

      let select = document.createElement("select");
      select.setAttribute("class", "item");
      select.setAttribute("id", `select${order}`);

      let option = document.createElement("option");
      option.setAttribute("value", "{}");
      let text = document.createTextNode("Select measurement");
      option.appendChild(text);
      select.appendChild(option);

      for (let i = 0; i < this.options.length; i++) {
        const measurement = this.options[i];
        option = document.createElement("option");
        option.setAttribute("value", `{${measurement.title}}`);
        // default select
        if (defaultSelectTitle && defaultSelectTitle == measurement.title)
          option.setAttribute("selected", "");
        text = document.createTextNode(measurement.title);
        option.appendChild(text);
        select.appendChild(option);
      }

      this.addElementToFormula(select);
      this.cursorMove("right");
      this.addEvent();
    },

    getFormulaString() {
      let element = document.querySelector("#vueFormulize");
      let child = element.childNodes;
      let arr = [...child];
      let str = "";

      for (let i = 0; i < arr.length; i++) {
        const xx = arr[i];
        if (xx.value) str += `${xx.value}`;
        if (!xx.value) str += `${xx.innerText}`;
      }

      this.output = str;
    },

    onFocus() {
      let element = document.getElementById("formula");
      element.focus();
    },

    onBlur() {
      this.getFormulaString();
      this.removeCursor();
      this.isActive = false;
      document.removeEventListener("keydown", this.captureKey);
      document.removeEventListener("paste", this.setClipboard);
    },

    setClipboard(e) {
      let clipboardData = e.clipboardData || window.clipboardData;
      let pastedData = clipboardData.getData("Text");
      this.pasteString(pastedData);
    },

    pasteString(string) {
      this.setFormula(string);
    },

    setFormula(stringFormula) {
      this.addEvent();
      let str = stringFormula;
      let measurement = "";
      let isMeasurement = false;

      for (let i = 0; i < str.split("").length; i++) {
        const item = str.split("")[i];
        if (!isMeasurement) {
          this.capturedKey = { key: item };
          this.updateFormula();
        }
        if (item == "{") {
          isMeasurement = true;
          continue;
        }
        if (item == "}") {
          isMeasurement = false;
          this.addMeasurement(measurement);
          continue;
        }
        if (isMeasurement) measurement += item;
      }
    },

    validation() {
      this.setStatus(null);
      this.errorName == null;

      let openBrackets = null;
      let closedBrackets = null;

      let str = this.output;
      let arr = str.split("");

      if (this.operators.includes(str[0])) {
        this.setStatus(
          "error",
          "The first element cannot start with the mathematical operator."
        );
        return;
      }

      for (let i = 0; i < arr.length; i++) {
        const item = arr[i];
        let afterItem = arr[i + 1];
        let beforeItem = arr[i - 1];
        if (item == "(") openBrackets++;
        if (item == ")") closedBrackets++;

        if (item == "{" && afterItem == "}") {
          this.setStatus("error", "Please select measurement");
          return;
        }

        // controle operators
        if (this.operators.includes(item)) {
          if (
            (afterItem && this.operators.includes(afterItem)) |
            (beforeItem && this.operators.includes(beforeItem))
          ) {
            this.setStatus(
              "error",
              "Mathematical operations cannot be sequenced."
            );
          }
          if (afterItem && this.specials.slice(2, 4).includes(afterItem)) {
            this.setStatus("error", "Please check mathematical operators.");
          }
          if (beforeItem && this.specials.slice(0, 2).includes(beforeItem)) {
            this.setStatus("error", "Please check mathematical operators.");
          }
        }

        // specials controller
        // for ), }
        if (this.specials.slice(2, 4).includes(item)) {
          if (afterItem && this.specials.slice(0, 2).includes(afterItem)) {
            this.setStatus(
              "error",
              "A new parentheses cannot be opened immediately after the closed parentheses."
            );
          }
          if (beforeItem && this.operators.includes(beforeItem)) {
            this.setStatus(
              "error",
              "Mathematical operator cannot be used just before the closed parenthesis."
            );
          }
          if (beforeItem && this.specials.slice(0, 2).includes(beforeItem)) {
            this.setStatus(
              "error",
              "Parentheses cannot be opened just before closed parentheses."
            );
          }
          if (afterItem && this.keys.includes(afterItem)) {
            this.setStatus(
              "error",
              "The number cannot be used immediately after the closed parenthesis."
            );
          }
        } // for (, {
        if (this.specials.slice(0, 2).includes(item)) {
          if (afterItem && this.operators.includes(afterItem)) {
            this.setStatus(
              "error",
              "The mathematical operator cannot be used immediately after the parentheses."
            );
          }
          if (beforeItem && this.specials.slice(2, 4).includes(beforeItem)) {
            this.setStatus(
              "error",
              "The mathematical operator cannot be used immediately before the closed parentheses."
            );
          }
          if (beforeItem && this.keys.includes(beforeItem)) {
            this.setStatus(
              "error",
              "The keys cannot be used immediately before the closed parentheses."
            );
          }
          if (afterItem && this.specials.slice(2, 4).includes(afterItem)) {
            this.setStatus("error", "Please do not leave the braces empty.");
          }
        }
      }
      if (openBrackets != closedBrackets) {
        if (openBrackets > closedBrackets) {
          this.setStatus(
            "error",
            "Please check the brackets. More than one bracket opened."
          );
          this.errorName = "bracket";
        }
        if (openBrackets < closedBrackets) {
          this.setStatus(
            "error",
            "Please check the brackets. More than one bracket closed."
          );
          this.errorName = "bracket";
        }
      } else {
        if (this.errorName != "measurement") {
          this.errorName = null;
          this.setStatus(null);
        }
      }
      if (!this.output) {
        this.setStatus(null);
      }
    }
  },

  mounted() {
    if (this.input) this.setFormula(this.input);
  },

  beforeUpdate() {
    if (this.error.type && this.error.message) {
      this.setStatus(this.error.type, this.error.message);
    } else {
      return null;
    }
  },

  beforeMount() {
    document.addEventListener("click", this.documentClick);
  },

  beforeDestroy() {
    document.removeEventListener("keydown", this.captureKey);
    document.removeEventListener("click", this.documentClick);
    document.removeEventListener("paste", this.setClipboard);
  }
};
</script>

<style>
* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
input[id="formula"] {
  opacity: 0;
  position: absolute;
  text-indent: -10000px;
  z-index: -20;
  outline: none;
}
.formula-cursor {
  display: inline-block;
  vertical-align: middle;
  height: 22px;
  width: 2px;
  margin: 0 1px;
  background-color: transparent;
  outline: none;
  border: 0;
  padding: 0 !important;
  border-right: 3px solid #000000;
  animation: blink 0.5s ease alternate infinite;
}
@keyframes blink {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
.vue-formulize-grid {
  padding: 0 1rem;
  border: 2px solid #d1d5da;
  background: #ffffff;
}
.vue-formulize-grid:hover {
  background: #f5f5f5;
}
.vue-formulize-grid:active {
  background: #ffffff;
}
.vue-formulize-grid.error {
  border-color: #ff5959;
}
.vue-formulize {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  flex-direction: row;
  flex-flow: row;
  overflow: auto;
  flex-wrap: nowrap;
  font-size: 1.5rem;
  line-height: initial;
  padding: 1rem;
  outline: none;
  border-radius: 5px;
  min-height: 126px;
  overflow: auto;
  max-width: 100%;
  cursor: pointer;
}

.vue-formulize.error-bracket .item {
  border-radius: 4px;
}
.vue-formulize.error-bracket .item[value="("] {
  background: rgba(255, 255, 0, 1);
}
.vue-formulize.error-bracket .item[value=")"] {
  background: rgba(255, 255, 0, 1);
}
.vue-formulize.error-measurement select {
  border-radius: 4px;
  background-color: rgba(255, 255, 0, 1) !important;
}
.vue-formulize .item {
  padding: 0px 0.1rem;
  background: transparent;
  outline: 0;
  border: 0;
  color: #000000;
  font-size: 1.2rem;
}
.vue-formulize .item.secondary {
  opacity: 0.8;
  font-size: 2.5rem;
}
.vue-formulize .item.space-m {
  margin: 0 0.5rem;
}
.vue-formulize .item.operator {
  width: 24px;
  height: 24px;
  font-size: 1.4rem;
  border-radius: 1rem;
  text-align: center;
  font-weight: 700;
  color: #000000;
  display: flex;
  align-items: center;
  justify-content: center;
}
.formula-action {
  outline: none;
  background-color: #ffffff;
  border: 1px dashed #a0a0a0;
  color: #a0a0a0;
  width: 28px;
  height: 28px;
  border-radius: 8px;
  display: flex;
  align-self: center;
  justify-content: center;
  cursor: pointer;
  text-align: center;
  line-height: initial;
  font-size: 18px;
}
.formula-action:hover {
  border: 1px dashed #238be6;
  color: #238be6;
}
.formula-item {
  width: 32px;
  height: 48px;
  border-radius: 0.5rem;
  display: flex;
  align-self: center;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: relative;
}
.formula-item::before {
  content: attr(data-text);
  font-size: 2rem;
  text-align: center;
  line-height: initial;
  margin: auto;
  top: 0;
  position: absolute;
}

.vue-formulize-error {
  font-size: 16px;
  line-height: initial;
  text-align: right;
}
.vue-formulize-error[type="error"] {
  color: #ff5959;
}
.vue-formulize-error[type="success"] {
  color: #15bb55;
}

select {
  border: 0;
  display: inline-block;
  outline: none;
  cursor: pointer;
  margin: 0 0.5rem;
  font-size: 15px !important;
  font-weight: 700;
  line-height: 1;
  vertical-align: baseline;
  border-radius: 4px;
  padding: 4px 8px !important;
  min-height: 36px;
  color: #434343;
  background-position: calc(100% - 0.5rem), center !important;
  background-size: 7% !important;
  padding-right: 2rem !important;
  /* background-image: url("~static/icons/arrow-down.svg") !important; */
  background-color: #ebebeb !important;
  background-repeat: no-repeat !important;
}
select:active {
  outline: 0;
  box-shadow: 0 0 0 2px #238be6;
}
select:-moz-focusring {
  color: transparent;
  text-shadow: 0 0 0 #000;
}
select option {
  font-size: 13pt;
  line-height: 200%;
  padding: 1rem;
  color: #000000;
  background: #ffffff;
  border: 0;
  outline: none;
}
</style>