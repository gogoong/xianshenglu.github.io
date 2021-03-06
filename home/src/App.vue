<template>
  <main id="app">
    <UnitLang :langs="authorData.langs" @langChange="langChangeHandler" />
    <UnitNav :navs="sections" @navChange="navChangeHandler"/>
    <Index :index="findSection('index')" id="index"/>
    <Introduction :introduction="findSection('introduction')"  id="introduction"/>
    <Skill :skill="findSection('skill')" id="skill"/>
    <Project :project="findSection('project')" id="project" />
  </main>
</template>

<script>
import authorData from './assets/js/authorData.js'
import UnitLang from './components/UnitLang'
import UnitNav from './components/UnitNav'
import Index from './components/Index'
import Introduction from './components/Introduction'
import Skill from './components/Skill'
import Project from './components/Project'

export default {
  name: 'App',
  components: {
    UnitLang,
    Index,
    Introduction,
    Skill,
    Project,
    UnitNav
  },
  data: function () {
    return {
      currentNavId: 'index',
      animationFrameId: 1,
      authorData
    }
  },
  computed: {
    sections () {
      return this.authorData.langs.find(lang => lang.checked).sections
    }
  },
  beforeCreate () {
    authorData.langs[1].checked = 'checked'
  },
  created () {
    this.langBackToLast()
  },
  mounted () {
    this.$nextTick(function () {
      this.scrollToLastPos()
      window.addEventListener('scroll', this.scrollHandlerGlobal, { capture: true }, true)
    })
  },
  updated () {

  },
  methods: {
    closest (ele, selector) {
      let targetEle = Array.from(document.querySelectorAll(selector))
      while (ele.tagName.toLowerCase() !== 'html') {
        if (targetEle.includes(ele)) {
          return ele
        }
        ele = ele.parentNode
      }
    },
    smoothScoll (targetScrollTop) {
      let self = this
      let speed = 0.1
      let startTime = 0
      let timeLimit = 2000
      window.cancelAnimationFrame(self.animationFrameId)
      self.animationFrameId = window.requestAnimationFrame(step)
      function step (timestamp) {
        startTime = startTime || timestamp
        let interval = timestamp - startTime
        let distance = targetScrollTop - document.documentElement.scrollTop
        let stepSize = speed * distance
        switch (Math.ceil(stepSize)) {
          case 0:
            stepSize = -1
            break
          case 1:
            stepSize = 1
            break
        }

        if (distance !== 0 && interval < timeLimit) {
          window.scrollTo(0, document.documentElement.scrollTop + stepSize)
          self.animationFrameId = window.requestAnimationFrame(step)
        }
      }
    },
    findSection (id) {
      return this.sections.find(section => section.id === id)
    },
    langClear () {
      this.authorData.langs.forEach(lang => { lang.checked = '' })
    },
    langBackToLast () {
      this.langClear()
      let lastLangId = localStorage.getItem('lang')

      if (lastLangId) {
        this.authorData.langs.find(lang => lang.id === lastLangId).checked = 'checked'
      } else {
        this.authorData.langs[0].checked = 'checked'
      }
    },
    langChangeHandler (e) {
      this.langClear()
      let preferredLangId = e.target.getAttribute('data-preferred-lang-id')
      localStorage.setItem('lang', preferredLangId)
      let preferredLang = this.authorData.langs.find(lang => lang.id === preferredLangId)
      preferredLang.checked = 'checked'
      this.syncNavBetweenLangs()
    },
    syncNavBetweenLangs () {
      this.navClear()
      this.sections.find(section => section.id === this.currentNavId).checked = 'checked'
    },
    navClear () {
      this.sections.forEach(section => { section.checked = '' })
    },
    navUpdate (targetNavId) {
      this.currentNavId = targetNavId
      this.sections.find(section => section.id === targetNavId)
        .checked = 'checked'
    },
    navChangeHandler (e) {
      this.navClear()
      let targetNavId = this.closest(e.target, '[data-target-nav-id]').getAttribute('data-target-nav-id')
      this.navUpdate(targetNavId)
      /*
      * document.documentElement.scrollTop in targetScrollTop will get the latest scrollTop
      * which will be more accurate than this.currentHtmlScrollTop,
      * especially when user doubleclicks the nav
      */
      let targetScrollTop = document.getElementById(targetNavId).getBoundingClientRect().top + document.documentElement.scrollTop
      /*
      TODO: {behavior: 'smooth'} is supported by Chrome and FF while IE and Edge don't support.
      TODO: That's why I made smoothScoll in methods.
      */
      // window.scrollTo({
      //   top: pos,
      //   behavior: 'smooth'
      // })
      this.smoothScoll(targetScrollTop)
    },
    scrollHandlerGlobal () {
      localStorage.setItem('currentHtmlScrollTop', document.documentElement.scrollTop)
      this.scrollHandlerForSkill()
      this.scrollHandlerForNav()
    },
    scrollHandlerForSkill () {
      let skillBoundingClientTop = document.getElementById('skill').getBoundingClientRect().top
      let viewportHeight = document.documentElement.clientHeight
      let isInView = !!(skillBoundingClientTop >= -viewportHeight / 1.3 && skillBoundingClientTop <= viewportHeight / 2)
      showSkillPercent(isInView)

      function showSkillPercent (isInView) {
        let circleAction = Array.from(
          document.querySelectorAll('circle[data-percent]')
        )
        circleAction.forEach(ele => {
          let percent = ele.getAttribute('data-percent')
          let stroke = ele.getAttribute('data-stroke')
          ele.style.strokeDasharray = isInView ? percent + ' 100' : ''
          ele.style.stroke = stroke
          ele.parentNode.parentNode.style.color = stroke
        })
      }
    },
    scrollHandlerForNav () {
      this.navClear()
      let scrollPos = Math.round(document.documentElement.scrollTop / document.documentElement.clientHeight)
      this.currentNavId = this.sections[scrollPos].id
      this.sections[scrollPos].checked = 'checked'
    },
    scrollToLastPos () {
      let lastPos = Number(localStorage.getItem('currentHtmlScrollTop'))
      if (lastPos) {
        document.documentElement.scrollTop = lastPos
      } else {
        this.sections[0].checked = 'checked'
      }
    }
  },
  destoryed () {
    window.removeEventListener('scroll', this.scrollHandlerGlobal, true)
  }
}

</script>

<style>
#app {
  position: relative;

  box-sizing: border-box;
  width: 100%;
  height: 100%;

  text-align: center;

  color: #2c3e50;

  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

</style>
