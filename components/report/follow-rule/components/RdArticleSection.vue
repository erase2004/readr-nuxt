<script>
/* global ScrollMagic */
/* eslint no-undef: "error" */
import RdReportArticle from '~/components/app/Report/RdReportArticle.vue'
import RdFullSlides from '~/components/report/follow-rule/components/RdFullSlides.vue'
import RdSlideCard from '~/components/report/follow-rule/components/RdSlideCard.vue'

export default {
  components: {
    RdReportArticle,
  },
  props: {
    sectionArticle: {
      type: Array,
      default: () => [],
    },
    loadScrollMagicScriptTimes: {
      type: Number,
      default: 0,
    },
    triggerHook: {
      type: Number,
      default: 0.2,
    },
  },
  data() {
    return {
      contentGroup: [],
      hasSendGa: false,
    }
  },

  computed: {
    viewportHeight() {
      return this.$store.getters['viewport/viewportHeight']
    },
  },

  watch: {
    loadScrollMagicScriptTimes(times) {
      if (times !== 4) return
      const controller = new ScrollMagic.Controller()
      new ScrollMagic.Scene({
        triggerElement: '.report-article__picture',
        triggerHook: 1,
        duration: `100px`,
      })
        .on('enter leave', (e) => {
          if (!this.hasSendGa) {
            this.$ga.event('projects', 'scroll', '圖1')
            this.hasSendGa = true
          }
        })
        // .addIndicators() // add indicators (requires plugin)
        .addTo(controller)
    },
  },

  mounted() {
    this.groupContent(this.splitUnderline(this.sectionArticle))
  },

  methods: {
    splitUnderline(contents) {
      contents = contents.map((content) => {
        if (typeof content.value !== 'string') return content
        const underlineContent = content.value.match(/<u*.*>*.*<\/u>/)
        if (underlineContent) {
          const newContent =
            '<u>' +
            underlineContent[0]
              .split('')
              .slice(3, underlineContent.length - 5)
              .join('</u><u>') +
            '</u>'
          content.value = content.value.replace(/<u*.*>*.*<\/u>/, newContent)
        }
        return content
      })
      return contents
    },
    sendGaEvent({ action, label }) {
      this.$ga.event('projects', action, label)
    },
    groupContent(contents) {
      let contentStore = []
      contents.forEach((content) => {
        if (content.type === 'fullSlides' || content.type === 'cards') {
          if (contentStore[0]) {
            this.contentGroup.push({ type: 'normal', content: contentStore })
            contentStore = []
          }
          this.contentGroup.push({ type: 'custom', content: [content] })
        } else {
          contentStore.push(content)
        }
      })
      if (contentStore[0])
        this.contentGroup.push({ type: 'normal', content: contentStore })
    },
    toggleFull(type) {
      this.$emit('toggleFull', type)
    },
    buildContent(content) {
      if (content.type === 'normal') {
        return (
          <RdReportArticle
            contents={content.content}
            slug="follow-rule"
            progressBarHeight="progressBarHeight"
            loadScrollMagicScriptTimes="loadScrollMagicScriptTimes"
            isPart={true}
            sendGaEvent={() => this.sendGaEvent}
          />
        )
      }

      const contentValue = content.content[0]
      switch (contentValue.type) {
        case 'fullSlides': {
          return (
            <RdFullSlides
              slides={contentValue.value}
              loadScrollMagicScriptTimes={this.loadScrollMagicScriptTimes}
              triggerHook={this.triggerHook}
              toggleFull={this.toggleFull}
            />
          )
        }

        default: {
          return (
            <RdSlideCard
              cards={contentValue.value}
              loadScrollMagicScriptTimes={this.loadScrollMagicScriptTimes}
              triggerHook={this.triggerHook}
              toggleFull={this.toggleFull}
            />
          )
        }
      }
    },
  },

  render() {
    return (
      <article>
        <div>{this.contentGroup.map(this.buildContent)}</div>
      </article>
    )
  },
}
</script>

<style lang="scss" scoped>
.article {
  padding: 48px 20px;
  color: #000000;
  background-color: rgba(254, 234, 223, 1);
  @include media-breakpoint-up(md) {
    padding: 48px 100px;
  }
  @include media-breakpoint-up(xl) {
    padding: 60px 100px;
  }
}
.report-article::v-deep {
  padding: 0 !important;
  a {
    color: rgba(40, 221, 177, 1);
  }

  .paragraph-with-annotation {
    .toggle {
      background: #ffffff;
      border: 1px solid rgba(40, 221, 177, 1);

      path {
        fill: #28ddb1;
      }
    }

    .annotation {
      background: #aeeee6;
      border: 1px solid rgba(0, 0, 0, 1);
    }
  }

  .container {
    > :first-child {
      margin-top: 48px !important;
    }

    > :last-child {
      margin-bottom: 0 !important;
    }
  }
}
</style>
