<template>
    <div class="main__conspekt">
        <div class="main-titles">
            <div class="main-titles__title" @click="toggleShow" :class="{ rotated: this.showed }">Содержание</div>
            <ul class="main-titles__list" :class="{'main-titles__list_showed': this.showed }">
                <li v-for="(title, index) in this.titles" :key="title.id" class="main-titles__item">
                    <a :href="'#heading' + (index + 1)" @click.prevent="toId"> <span>{{index + 1}}. </span>
                        <span>{{title.heading}}</span> </a>
                    <ul class="main-titles__sublist" v-if="title.subHeadings.length">
                        <li v-for="(point, subIndex) in title.subHeadings" :key="point.id" class="main-titles__subitem">
                            <a :href="'#subHeading' + (index+1) + '.' + (subIndex +1) " @click.prevent="toId">
                                <span class="subHeading__number">  {{index + 1}}.{{subIndex + 1}}</span>
                                &nbsp;
                                <span>{{ point}}</span>
                            </a>
                        </li>
                    </ul>
                </li>
            </ul>
        </div>
        <div class="main-information">
            <section v-for="(section) in this.textsWidthId" :key="section.id" class="main-information__section">
                <div v-html="section.heading"></div>
                <div v-for="part in section.parts" :key="part.id">
                    <div v-html="part.title"></div>
                    <div v-html="part.text"></div>
                </div>
            </section>
        </div>

    </div>
</template>

<script>
export default {
  name: "lessonText",
  props: {
    text: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      showed: true
    };
  },
  methods: {
    getHeadingsIndex(text, target) {
      let pos = -1,
        arr = [];
      while ((pos = text.indexOf(target, pos + 1)) != -1) {
        arr.push(pos);
      }
      return arr;
    },
    concatHeadingIndexes(fir, sec) {
      return fir.reduce(
        (acc, item, index) => [...acc, Array.from(new Set([item, sec[index]]))],
        []
      );
    },
    sliceText(text, indexes) {
      let arr = [];
      indexes.forEach(ind => {
        arr.push(text.slice(ind[0], ind[1]));
      });
      return arr;
    },
    cleanHeadings(headings, sel) {
      let arr = [];
      headings.map(heading => arr.push(heading.split(sel)));
      return arr;
    },
    deleteEmpty(arr) {
      let a = [];
      arr.map(item => a.push(item[1].trim()));
      return a;
    },
    getParts(sections, text) {
      let arr = [];
      for (let i = 0; i < sections.length; ++i) {
        if (sections[i + 1]) {
          let newText = text.slice(sections[i][0], sections[i + 1][0]);
          arr.push(newText);
        } else if (!sections[i + 1]) {
          let newText = text.slice(sections[i][0], text.length);
          arr.push(newText);
        }
      }
      return arr;
    },
    getTitles(textes, target, elems) {
      let global = [];
      textes.forEach(text => {
        let headingStart = this.getHeadingsIndex(text, target[0][0]),
          headingEnd = this.getHeadingsIndex(text, target[0][1]),
          subHeadingStart = this.getHeadingsIndex(text, target[1][0]),
          subHeadingEnd = this.getHeadingsIndex(text, target[1][1]);
        let headingsIndexes = this.concatHeadingIndexes(
            headingStart,
            headingEnd
          ),
          subHeadingsIndexes = this.concatHeadingIndexes(
            subHeadingStart,
            subHeadingEnd
          );
        let headingCode = this.sliceText(text, headingsIndexes),
          subHeadingsCode = this.sliceText(text, subHeadingsIndexes);
        let obj = Object.create(null);
        obj.heading = this.deleteEmpty(
          this.cleanHeadings(headingCode, elems[0])
        )[0];
        obj.subHeadings = this.deleteEmpty(
          this.cleanHeadings(subHeadingsCode, elems[1])
        );
        global.push(obj);
      });
      return global;
    },

    getText(textes, target) {
      let global = [],
        blocks = [],
        parts = [];
      textes.forEach(text => {
        let headingStart = this.getHeadingsIndex(text, target[0][0]),
          headingEnd = this.getHeadingsIndex(text, target[0][1]),
          subHeadingStart = this.getHeadingsIndex(text, target[1][0]),
          subHeadingEnd = this.getHeadingsIndex(text, target[1][1]);
        let headingsIndexes = this.concatHeadingIndexes(
            headingStart,
            headingEnd
          ),
          subHeadingsIndexes = this.concatHeadingIndexes(
            subHeadingStart,
            subHeadingEnd
          );
        let headingCode = this.sliceText(text, headingsIndexes),
          parts;
        if (subHeadingStart.length && subHeadingEnd.length) {
          parts = this.getParts(subHeadingsIndexes, text);
        } else {
          parts = this.getParts(
            [headingStart, [headingEnd[0] - headingStart[0]]],
            text
          );
          parts[0] = parts[1]
            .trim()
            .slice(5, parts[1].length)
            .trim();
          parts.length = 1;
        }
        let obj = Object.create(null);
        obj.heading = headingCode.map(h => h + target[0][1]);
        obj.parts = parts;

        global.push(obj);
      });
      global.map(obj => parts.push(obj.parts));
      parts.map(part => {
        let cleanText = [];
        part.forEach(string => {
          let str = string.slice(
            this.getHeadingsIndex(string, target[1][1]),
            string.length
          );
          let cleanStr = str
            .trim()
            .slice(5, str.length)
            .trim();
          let title =
            string.slice(
              this.getHeadingsIndex(string, target[1][0]),
              this.getHeadingsIndex(string, target[1][1])
            ) + target[1][1];
          cleanText.push({
            title: title,
            text: cleanStr
          });
        });
        blocks.push(cleanText);
      });
      global.forEach((elem, index) => {
        elem.parts = blocks[index];
      });
      return global;
    },

    getTextWithId(textes) {
      let arr = [];
      textes.forEach((el, index) => {
        let titleStart = el.heading[0].slice(0, 3) + " ";
        let id = `id="heading${index + 1}"`;
        let titleMid = el.heading[0].slice(3, 4);
        let titleInd = index + 1 + "." + "  ";
        let titleEnd = el.heading[0].slice(4, el.heading[0].length);
        el.heading = titleStart + id + titleMid + titleInd + titleEnd;
        let parts = el.parts;
        parts.forEach((part, point) => {
          part.title =
            part.title.slice(0, 3) +
            " " +
            `id="subHeading${index + 1}.${point + 1}"` +
            part.title.slice(3, 4) +
            (index + 1) +
            "." +
            (point + 1) +
            "." +
            "  " +
            part.title.slice(4, part.title.length);
        });
        arr.push(el);
      });
      return arr;
    },

    toId(e) {
      let target =
        e.target.getAttribute("href") ||
        e.target.parentNode.getAttribute("href");
      let targetId = target.slice(1, target.length);
      let elem = document.getElementById(targetId);
      let top = elem.getBoundingClientRect().top;
      let windowScroll = window.screenY;
      let h = parseInt(getComputedStyle(document.querySelector(".top")).height);
      if (windowScroll < top) {
        document.documentElement.scrollTop += top - h;
      }
    },
    toggleShow() {
      this.showed = !this.showed;
    },
    getPartSingle(i) {
      return this.parts[i];
    }
  },
  computed: {
    sectionStartIndexes() {
      return this.getHeadingsIndex(this.text, "<h2");
    },
    sectionEndIndexes() {
      return this.getHeadingsIndex(this.text, "</h2>");
    },
    sectionIndexes() {
      return this.concatHeadingIndexes(
        this.sectionStartIndexes,
        this.sectionEndIndexes
      );
    },
    parts() {
      return this.getParts(this.sectionIndexes, this.text);
    },
    titles() {
      return this.getTitles(
        this.parts,
        [["<h2", "</h2>"], ["<h3", "</h3>"]],
        ["<h2>", "<h3>"]
      );
    },
    texts() {
      return this.getText(this.parts, [["<h2", "</h2>"], ["<h3", "</h3>"]]);
    },
    textsWidthId() {
      return this.getTextWithId(this.texts);
    }
  }
};
</script>

<style lang="sass">
    @import "../globalStyles/variables"
    .main__conspekt
        width: 98%
        margin-left: auto
        margin-right: auto
        padding-left: 105px
        padding-right: 105px
        font-family: $basic
        font-size: 16px
        color: #1D1D1D
        letter-spacing: 0
        line-height: 26px

    .subHeading__number
        font-family: $bold

    .main-titles
        width: 100%
        background-color: #F9FAFC
        border-radius: 4px
        padding: 17px 14px 31px 14px
        margin-top: 22px
        margin-bottom: 39px
        a
            color: inherit
        &__list
            transition: .3s ease
            max-height: 0
            overflow: hidden
            Li:first-of-type
                padding-top: 18px
            &_showed
                max-height: 10000px
        &__title
            font-family: $bold
            font-size: 16px
            color: #41454D
            letter-spacing: 0
            width: 100%
            padding: 0 12px 14px 12px
            border-bottom: 1px solid #EAEEEF
            position: relative
            cursor: pointer
            &:after
                content: ''
                display: block
                position: absolute
                right: 12px
                top: calc(50% - 12px)
                background: transparent url("../assets/arrowDown.svg") no-repeat center center
                background-size: contain
                width: 18px
                height: 12px
                opacity: .8
                transition: .3s ease-in-out
                transform: rotate(180deg)
            &.rotated
                &:after
                    transform: none
        &__item
            font-family: $bold
            font-size: 13px
            color: #41454D
            line-height: 21px
            margin-bottom: 15px
            &:before
                margin-right: 8px
        &__subitem
            font-family: $basic
            margin-bottom: 4px
            &:before
                font-family: $bold
                font-size: 13px
                color: #41454D
                line-height: 21px
                margin-right: 8px
        &__sublist
            padding-left: 20px

    .main-information
        padding-top: 44px
        border-top: 1px solid #f5f5f5
        padding-bottom: 70px
        h2
            font-family: $bold
            font-size: 20px
            color: #1D1D1D
            letter-spacing: 0
            line-height: 24px
            margin-bottom: 25px
        h3
            font-family: $bold
            font-size: 18px
            color: #1D1D1D
            letter-spacing: 0
            line-height: 24px
            margin-bottom: 20px
        img
            display: block
            margin: 25px auto
        strong
            font-family: $bold
        &__section
            margin-bottom: 66px
            &:last-of-type
                margin-bottom: 0
</style>
