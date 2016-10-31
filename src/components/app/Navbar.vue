<template>
  <div class="navbar">
    <div v-if="showBackButton" class="back-button" @click="backButtonClicked()" transition="fade">
      {{{ backButtonText }}}
    </div>

    <div v-if="showMenuButton" class="menu-button" @click="menuButtonClicked()" transition="fade">
      {{{ menuButtonText }}}
    </div>

  </div>
</template>
<style lang='scss'>
  @import "../scss/variables";
  @import '../scss/mixins';
  $themeColor: '#007aff';
  $ios-transition-timing-function: ease;
  $android-transition-timing-function: linear;

  .navbar {
    box-sizing: border-box;
    -webkit-tap-highlight-color: rgba(0,0,0,0);
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 44px;
    z-index: 10;
    background-color: #fff;
    @include hairline(bottom);

    .back-button, .menu-button {
      position: absolute;
      top: 0;
      width: 80px;
      height: 44px;
      line-height: 44px;
      z-index: 12;

      span {
        font-size: 16px;
        line-height: 20px;
      }

      i.icon {
        line-height: 20px;
      }
    }

    .back-button {
      left: 0;
      padding: 0 0 0 10px;
      text-align: left;
    }

    .menu-button {
      right: 0;
      padding: 0 10px 0 0;
      text-align: right;
    }

    .center {
      position: absolute;
      top: 0px;
      left: 0;
      width: 100%;
      height: 44px;
      padding: 0;
      text-align: center;
      z-index: 11;

      .title {
        display: inline-block;
        font-size: 18px;
        line-height: 44px;
        @include transition-duration($ios-transition-duration);
        @include transition-timing-function($ios-transition-timing-function);
        -webkit-transition-property: opacity, -webkit-transform, box-shadow;
        transition-property: opacity, transform, box-shadow;
      }
    }

    .fade-transition {
      @include transition-duration($android-transition-duration);
      @include transition-timing-function($android-transition-timing-function);
      opacity: 1;
    }

    .fade-enter, .fade-leave {
      opacity: 0;
    }
  }

  // android or other
  .grade-b .navbar {
    .center .title,
    .fade-transition
    {
      @include transition-property(all);
      @include transition-duration($android-transition-duration);
      @include transition-timing-function($android-transition-timing-function);
    }
  }

</style>
<script>
  import channel from './channel'
  import utils from './utils'

  function getTitleTransitionDistance(t) {
    return (document.body.offsetWidth - t.offsetWidth) / 2 - 10
  }

  function centerElement(navbar, title, direction) {
    let centerId = Math.random().toString(36).substring(3, 8)
    let c = document.createElement('div')
    c.id = centerId
    c.className = 'center'
    let t = document.createElement('span')
    t.className = 'title'
    t.innerHTML = title

    let reverse = direction == 'back'
    t.style.opacity = 0

    if (utils.is_ios_device()) {
      t.style.transform = 'translate3d(' + (reverse ? '-' : '') + getTitleTransitionDistance(t) + 'px,0,0)'
      t.style.webkitTransform = 'translate3d(' + (reverse ? '-' : '') + getTitleTransitionDistance(t) + 'px,0,0)'
    }

    if (!navbar.querySelector('.center')) {
      t.style.opacity = 1
      t.style.transform = 'translate3d(0,0,0)'
      t.style.webkitTransform = 'translate3d(0,0,0)'
    }

    c.appendChild(t)
    navbar.appendChild(c)
    return document.getElementById(centerId)
  }

  function titleIn(t) {
    t.style.opacity = 1
    t.style.transform = 'translate3d(0,0,0)'
    t.style.webkitTransform = 'translate3d(0,0,0)'
  }

  function titleOut(t, direction) {
    let reverse = direction == 'back'
    t.style.opacity = 0
    if (utils.is_ios_device()) {
      t.style.transform = 'translate3d(' + (reverse ? '' : '-') + getTitleTransitionDistance(t) + 'px,0,0)'
      t.style.webkitTransform = 'translate3d(' + (reverse ? '' : '-') + getTitleTransitionDistance(t) + 'px,0,0)'
    }
  }

  export default {
    data() {
      return {
        title: '',
        showBackButton: false,
        onBackButtonClick: undefined,
        showMenuButton: false,
        onMenuButtonClick: undefined,
        backButtonText: '<i class="icon icon-back"></i>',
        menuButtonText: '<i class="icon icon-bars"></i>',
      }
    },

    created() {
      // center leave
      let cl;

      channel.$on('PageTransitionEvent', (data) => {
        let direction = document.querySelector('[von-app]').getAttribute('transition-direction')
        this.title = data.title

        let c = centerElement(this.$el, this.title, direction)

        setTimeout(() => {
          titleIn(c.querySelector('.title'))
          if (cl) {
            titleOut(cl.querySelector('.title'), direction)
          }

          setTimeout(() => {
            if (cl) this.$el.removeChild(cl)
            cl = c
          }, 500)
        })

        // nav item & click event handler
        this.showBackButton = data.showBackButton
        this.onBackButtonClick = data.onBackButtonClick
        if (data.backButtonText)
          this.backButtonText = data.backButtonText

        this.showMenuButton = data.showMenuButton
        this.onMenuButtonClick = data.onMenuButtonClick
        if (data.menuButtonText)
          this.menuButtonText = data.menuButtonText
      })
    },

    ready() {
    },

    methods: {
      backButtonClicked() {
        if (this.onBackButtonClick) {
          this.onBackButtonClick()
          return
        }

        $router.nextDirection('back')
        history.go(-1)
      },

      menuButtonClicked() {
        if (this.onMenuButtonClick) {
          this.onMenuButtonClick()
        }
      }
    }
  }

</script>