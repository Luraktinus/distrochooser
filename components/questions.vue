<template>
  <div class="column col-9 col-xs-12">
    <div class="columns">
      <div class="column col-7 col-xs-12" id="accordion">
        <div class="panel col-xs-12 show-xs mobile-header">
          <div class="panel-header text-center">
            <div class="mobile-flag-parent">
              <a class="mobile-flag" title="Zur deutschen Version wechseln" href="/">
                <img src="/de.png" alt="Deutsch">
              </a>
              <a class="mobile-flag" title="Switch to english version" href="/en/">
                <img src="/gb.png" alt="English">
              </a>
		
              <a class="mobile-flag" title="Switch to chinese (simplified) version" href="/zh-cn/">
                <img src="/zh-cn.png" alt="Chinese (simplified)">
              </a>

            </div>
            <figure class="avatar avatar-lg">
              <img src="/logo.png">
            </figure>
          </div>
          <nav class="panel-nav">
            <ul class="tab tab-block mobile-navigation">
              <li class="tab-item">
                <a :href="'/info/'+this.globals.lang+'/about'">
                  {{ text("sys.about.short") }}
                </a>
              </li>

              <li class="tab-item">
                <a href="https://blog.distrochooser.de">
                  Blog
                </a>
              </li>
              <li class="tab-item">
                <a :href="'/info/'+this.globals.lang+'/privacy'">
                  {{ text("sys.privacy") }}
                </a>
              </li>
              <li class="tab-item">
                <a :href="'/info/'+this.globals.lang+'/contact'">
                  {{ text("sys.contact") }}
                </a>
              </li>
            </ul>
          </nav>
        </div>
        <bridge></bridge>
        <lang></lang>
        <div class="accordion" :class="{'accordeon-disabled disabled': weigthActive || resultWayChoosed}">
          <div class="accordion-item" v-for="(q,qindex) in this.globals.distrochooser.questions" v-bind:key="q.id" >
            <input type="radio" :id="'header' + q.id" name="accordion-radio" hidden="" v-on:click="hideResults">
            <label class="accordion-header hand columns" :class="{'answered':q.answered}"  :for="'header' + q.id">
               <span class="col-1" v-if="q.number !== -1"> {{ qindex }}. </span>{{ q.title }}
               <i v-if="q.number !== -1 && q.answered" class="icon icon-check answered-check" ></i>
            </label>
            
            <div class="accordion-body">
              <div class="panel-body">
                <p class="question-text" v-html="q.number === -1 ? '' : q.text"></p>
                <div class="toast toast-warning exclusion-warning" v-if="q.excludedBy !== null && isTagMatchOnlyPositives(q.excludedBy)">
                  {{ text('sys.excludedbytag') }}
                </div>
                <div id="StartText" v-if="q.id === 'welcome'">
                  <p class="question-text" v-html="text('sys.intro')"></p>
                  <ul class="list">
                    <li>{{ text('sys.can-skip-questions') }} </li>
                    <li>{{ text('sys.can-get-result-anytime') }} </li>
                    <li>{{ text('sys.can-get-result-anyorder') }}</li>
                    <li v-html="text('sys.can-delete')"></li>
                    <li v-html="text('sys.can-mark-important')"></li>
                  </ul>
                </div>
                <div v-if="q.answers.length !== 0" class="answer-parent">
                  <div :class="q.isSingle ? 'radio' : 'checkbox'" v-for="(a,aindex) in q.answers" v-bind:key="a.id">
                    <p class="answer-input" v-if="q.isSingle">
                      <label class="form-radio" :for="a.id" v-bind:class="{ 'selected': a.selected }">
                        <input :id="a.id" :checked='a.selected ' :name="q.id + '_a'" :data-id="a.id" type="radio" v-on:click="answer(q, a)">
                        <i class="form-icon"></i> {{ a.text }}
                      </label>
                      <i v-on:click.prevent="showTooltip(translateExcludedTags(a),$event)" v-if="a.excludeTags.length > 0" class="fa fa-question-circle fa-question-exclude" data-placement='left' data-html="true" :data-title="translateExcludedTags(a)"></i>
                    </p>
                    <p class="answer-input" v-if="!q.isSingle">
                      <label class="form-checkbox" :for="a.id" v-bind:class="{ 'selected': a.selected }">
                        <input :id="a.id" :checked='a.selected ' :name="q.id + '_a'" :data-id="a.id" type="checkbox" v-on:click="answer(q, a)">
                        <i class="form-icon"></i> {{ a.text }}
                      </label>
                      <i v-on:click.prevent="showTooltip(translateExcludedTags(a),$event)" v-if="a.excludeTags.length > 0" class="fa fa-question-circle" data-placement='left' data-html="true" :data-title="translateExcludedTags(a)"></i>
                    </p>
                  </div>
                </div>
                <div class="btn-group btn-group-block answer-buttons" v-if="q.number !== -1 && lastQuestionNumber !== qindex">
                  <a v-on:click.prevent="nextTrigger(q)" class="btn"> <i class="icon icon-check"></i> {{ text("sys.next") }}</a>
                  <a v-if="!q.answered && lastQuestionNumber !== q.number && q.number !== -1" class="btn" v-on:click.prevent="nextTrigger(q)"> <i class="icon icon-cross"></i> {{ text("sys.skip") }} </a>
                  <a v-if="q.answered" class="btn danger" v-on:click.prevent="removeAnswers(q)"> <i class="icon icon-delete"></i> {{ text("sys.clear") }} </a>
                </div>

                <div class="btn-group btn-group-block" v-if="lastQuestionNumber === qindex">
                  <a v-if="q.answered" class="btn danger" v-on:click.prevent="removeAnswers(q)"> <i class="icon icon-delete"></i> {{ text("sys.clear") }} </a>
                </div>
                <a class="btn btn-primary btn-start" href="#" v-if="q.number === -1" v-on:click.prevent="nextTrigger(q)">
                  {{ text("sys.start") }}
                </a>
              </div>  
            </div>
          </div>
        </div>

        <!-- result part -->
        <div class="columns preresult" v-if="!resultWayChoosed && this.answered.length > 0 && !weigthActive">
          <h4 id="weighting" class="hide-xs"> {{ text("sys.weightorresult") }} </h4>
          <h4 id="weighting" class="show-xs"> {{ text("sys.weightorresult") }}</h4>
          <div class="column col-5">
            <a class="btn" v-on:click.prevent="toggleResult" href="#">{{ text("sys.getresult") }}</a>
          </div>    
          <div class="column col-2 or hide-xs">
            {{ text("sys.or") }}
          </div> 

          <div class="column col-1 or show-xs"></div> 
          <div class="column col-5">
            <a class="btn" href="#" v-on:click.prevent="toggleWeighting">{{ text("sys.weight") }}</a>
          </div>
        </div>
        <div class="weight-active" v-if="weigthActive">
          <h4>{{ text("sys.weight") }}</h4>
          <div class="toast toast-success weight-info">
            {{ text("sys.weightinfo") }}
          </div>
          <div class="form-group">
            <div v-for="(tag,key) in tags" v-bind:key="key" v-if="!tag.negative">
                <div>
                  {{ text(key) }} 
                  <span class="important" v-if="parseInt(tag.weight) > 1 ">{{ text("important") }}</span>
                  <span class="notimportant" v-if="parseInt(tag.weight) <1 ">{{ text("notimportant") }}</span>
                </div>
                <div class="columns">
                  <span class="column col-4 notimportant">{{ text('sys.notimportant') }}</span>
                  <div class="column col-4"> 
                    <input class="slider" type="range" min="0" max="2" step="1" value="1" v-model="tag.weight">
                  </div>
                  <span class="column col-4 important">{{ text('sys.important') }}</span>
                </div>
            </div>
          </div>
          <div class="toast toast-error weight-info" v-if="isDangerousTagCondition">
            {{ text("sys.weighterror") }}
          </div>
          <div class="btn-group columns" v-if="!isDangerousTagCondition">
            <a class="btn" v-on:click.prevent="toggleWeighting" href="#"> {{ text("sys.abort") }}</a>
            <a class="btn" v-on:click.prevent="toggleResult" href="#">{{ text("sys.getresult") }}</a>
          </div>
        </div>
        <div class="results" v-if="resultWayChoosed">
          <div class="columns">
            <a class="btn btn-primary centered back-button" v-on:click.prevent="toggleResult"> {{ text("sys.back") }} </a>
          </div>
          <distrolist :distros="distros" :parent="this"></distrolist>
        </div>
      </div>
      <div class="column col-2 hide-xs">
      </div>
      <button class="btn btn-primary show-xs mobile-result-button" v-if="this.answered.length > 0" :class="{'disabled': this.weigthActive || this.resultWayChoosed}" v-on:click.prevent="jumpToBottom">
        {{ text('sys.getresult') }}
      </button>
      <div class="column col-3 hide-xs right-box fixed">
          <div class="panel">
            <div class="panel-header text-center">
              <figure class="avatar avatar-lg">
                <img src="/logo.png">
              </figure>
              <div class="panel-subtitle">{{ answered.length + "/" + (this.globals.questions.length - 1) + " " + text('sys.answered') }}</div>
              <progress class="progress" :value="answered.length" :max="this.globals.questions.length - 1"></progress>
            </div>
            <div class="panel-body">
              <div class="form-group">
                <label class="form-switch">
                  <input type="checkbox" v-model="globals.distrochooser.options.displayExcluded">
                  <i class="form-icon"></i> {{ text('sys.displayexcluded') }}
                </label>
              </div>
              <div class="panel-footer">
                <button class="btn btn-primary" :class="{'disabled':this.answered.length === 0}" v-on:click.prevent="jumpToWeighting">{{ text('sys.getresult') }}</button>
              </div>
            </div>
          </div>
      </div>
    </div>
  </div>
</template>
 
<script>
import Vue from 'vue' // eslint-disable-line no-unused-vars
import i18n from '../mixins/i18n'
import nuxt from '../mixins/nuxt-wrapper'
import distrolist from './distrolist'
import bridge from './bridge'
import lang from './lang'
export default {
  data: function () {
    return {
      weigthActive: false,
      resultWayChoosed: false,
      tags: {},
      displayTest: -1
    }
  },
  mixins: [
    i18n,
    nuxt
  ],
  components: {
    distrolist,
    bridge,
    lang
  },
  created: function () {
    this.globals.mainInstance = this
  },
  mounted: function () {
    jQuery('#headerwelcome').trigger('click') // eslint-disable-line no-undef
  },
  computed: {
    isDangerousTagCondition: function () {
      if (!this.weigthActive) {
        return false
      }
      var negativeTags = 0
      var tagCount = 0
      for (var p in this.tags) {
        // the check is only for tags the user can weight. If all of the positive tags are "downweighted", we cannot calculate any results
        var isNegative = this.tags[p].negative === false && parseInt(this.tags[p].weight) === 0
        if (isNegative) {
          negativeTags++
        }
        tagCount++
      }
      return negativeTags === tagCount
    },
    lastQuestionNumber: function () {
      return this.globals.questions.length - 1
    },
    answered: function () {
      return this.globals.questions.filter(function (q) {
        return q.answered
      })
    },
    distros: function () {
      // var raw =  this.globals.distros
      var results = []
      var _t = this
      console.log(this.globals)
      this.globals.distros.forEach(function (d) {
        var hits = []
        var antihits = []
        d.results = {}
        for (var k in _t.tags) {
          /**
           * Case I : tag 'foo' -> tag (distro) 'foo'
           * Case II : tag 'foo' -> tag (distro) '!foo'
           * Case III : tag 'foo', negative
           * */
          if (d.tags.indexOf(k) !== -1 && hits.indexOf(k) === -1) {
            if (_t.tags[k].negative) {
              antihits.push(k)
            } else {
              hits.push(k)
            }
          }
          if (d.tags.indexOf('!' + k) !== -1 && antihits.indexOf(k) === -1) {
            antihits.push(k)
          }
        }
        var distroPoints = 0
        // calculate sum with weight
        hits.forEach(function (t) {
          var weight = _t.tags[t].weight
          var amount = _t.tags[t].amount // a tag can be given more than one times, causes "heavier" weight
          var sum = amount * weight
          distroPoints += sum
          d.results[t] = _t.tags[t]
        })
        // calculate sum with weight
        antihits.forEach(function (t) {
          /*
          var weight = _t.tags[t].weight
          var amount = _t.tags[t].amount // a tag can be given more than one times, causes "heavier" weight
          var sum = amount * weight
          */
          distroPoints = 0
          d.results[t] = _t.tags[t]
        })
        // calculate percentage
        d.points = distroPoints
        results.push(d)
      })
      return results.sort(function (a, b) {
        return a.points < b.points ? 1 : -1
      })
    }
  },
  methods: {
    isTagMatchOnlyPositives: function (tags) {
      for (var i = 0; i < tags.length; i++) {
        var tag = tags[i]
        if (typeof (this.tags[tag]) !== 'undefined' && !this.tags[tag].negative) {
          return true
        }
      }
      return false
    },
    isTagMatch: function (tags) {
      for (var i = 0; i < tags.length; i++) {
        var tag = tags[i]
        if (typeof (this.tags[tag]) !== 'undefined') {
          return true
        }
      }
      return false
    },
    computeTags: function () {
      var result = {}
      var _t = this
      for (var i = 0; i < this.answered.length; i++) {
        this.answered[i].answers.forEach(function (element) {
          if (element.selected) {
            var tag = element.tags
            tag.forEach(function (t) {
              if (typeof result[t] === 'undefined') {
                result[t] = {
                  amount: 1,
                  weight: typeof _t.tags[t] !== 'undefined' ? _t.tags[t].weight : 1,
                  negative: false
                }
              } else {
                result[t].amount++
              }
            })
            tag = element.excludeTags
            tag.forEach(function (t) {
              var name = t.replace('!', '')
              if (typeof result[name] === 'undefined') {
                result[name] = {
                  amount: 1,
                  weight: typeof _t.tags[t] !== 'undefined' ? _t.tags[t].weight : 1,
                  negative: true
                }
              } else {
                result[name].amount++
              }
            })
          }
        }, this)
      }
      this.tags = result
      this.hideResults()
    },
    nextTrigger: function (q) {
      var index = this.globals.questions.indexOf(q)
      if (index === this.lastQuestionNumber) { // eslint-disable-line space-infix-ops
        return
      }
      jQuery('#header' + q.id).trigger('click') // eslint-disable-line no-undef
      var next = this.globals.questions[index + 1]
      jQuery('#header' + next.id).trigger('click') // eslint-disable-line no-undef
      jQuery('#header' + next.id).animate({ scrollTop: jQuery('#header' + next.id).offset().top }, 10) // eslint-disable-line no-undef
      this.resultWayChoosed = false
      this.weigthActive = false
    },
    answer: function (q, a) {
      this.resultWayChoosed = false
      this.weigthActive = false
      this.globals.preloadInfos = null // we do not need them anymore if something is answered
      if (q.isSingle) {
        this.removeAnswers(q)
      }
      q.answered = false
      for (var i in q.answers) {
        // input[type='radio'] needs to be set per code that we can handle it the same way as input[type='checkbox']
        if (q.answers[i] === a) {
          q.answers[i].selected = !q.answers[i].selected
        }
        if (q.answers[i].selected) {
          q.answered = true
        }
      }
      this.computeTags()
    },
    removeAnswers: function (q) {
      for (var i in q.answers) {
        q.answers[i].selected = false
      }
      q.answered = false
    },
    toggleWeighting: function () {
      this.weigthActive = !this.weigthActive
      if (this.globals.preloadInfos !== null) {
        this.globals.preloadInfos.tags.forEach(function (tag) {
          this.tags[tag.name] = tag
        }, this)
        this.globals.preloadInfos = null
        this.weigthActive = true
        this.resultWayChoosed = false
      }
      if (this.weigthActive) {
        this.jumpToBottom()
      }
    },
    toggleResult: function () {
      if (this.weigthActive) {
        this.toggleWeighting()
      }
      this.resultWayChoosed = !this.resultWayChoosed
      if (this.resultWayChoosed) {
        if (this.globals.preloadInfos !== null) {
          this.computeTags()
          this.globals.preloadInfos = null
          this.weigthActive = false
          this.resultWayChoosed = true
        }
        this.globals.distrochooser.addResult(this)
      }
    },
    jumpToWeighting: function () {
      jQuery('html, body').animate({ scrollTop: jQuery('#weighting').offset().top }, 10) // eslint-disable-line no-undef
    },
    jumpToBottom: function () {
      jQuery('html,body').animate({scrollTop: document.body.scrollHeight}, 10) // eslint-disable-line no-undef
    },
    hideResults: function () {
      this.weigthActive = false
      this.resultWayChoosed = false
    }
  }
}
</script>
 
<style scoped>
  .answer-parent {
    margin-left: 1em;
  }
  .answer-parent input[type='checkbox'],.answer-parent input[type='radio']{
    margin-left: 0.1em;
  }
  .answered{
    font-weight: bold;
  }
  .selected{
    font-weight: bold;
  }
  .back-button{
    display: block;
    margin-bottom: 1em;
  }
  .right-box{
    right: 4em;
    max-width: 14%;
  }
  #weighting{
    display: block;
    width: 100%;
  }
  .exclusion-warning{
    margin-bottom: 0.6em;
  }
  .preresult{
    text-align: center;
    margin-bottom: 2em;
  }
  .or{
    font-size: large;
    font-weight: 400;
  }
  .notimportant{
    font-weight: 300;
    font-style: italic;
  } 
  .important{
    font-weight: bold;
  }
  .accordeon-disabled{
    opacity: 0.5;
  }
  .avatar{
    background: white !important;
  }
  .weight-info{
    margin-bottom: 0.8em;
  }
  .mobile-flag{
    margin-left: 1em;
  }
  .mobile-flag img{
    height: 1.5em;
  }
  .mobile-flag-parent{
    margin-top: -12px;
    text-align: left;
    margin-left: -27px;
  }
  .question-text{
    margin-top: .7em;
    margin-right: 1em;
    margin-left: 0.7em;
    font-size: 10.3pt;
    margin-bottom: 0.5em;
  }
  .accordion-header{
    background-color: #fbfafa;
    padding-top: .5em;
    padding-bottom: .5em;
    margin-left: 0.0em;
    margin-right: 0.0em;
  }
  .accordion .accordion-item input:checked~.accordion-body{
    border-left:2px solid #f8f9fa;
    border-right: 2px solid #f8f9fa;
    border-bottom: 2px solid #f8f9fa;
  }
  .accordion .accordion-item .accordion-body{      
    margin-bottom: .6rem;
  }
  .answered-check{
    margin-bottom: 0.3em;    
    margin-left: 0.3em;
    color: #32b643;    
    height: 100%;
    padding-bottom: .5em;
  }
  .accordion input:checked~.accordion-header .icon {
    transform: rotate(0deg) !important;
  }
  .mobile-result-button{
    position: fixed;
    top: 50%;
    right: -2.6em;
     transform:rotate(270deg);
    -ms-transform:rotate(270deg); /* IE 9 */
    -moz-transform:rotate(270deg); /* Firefox */
    -webkit-transform:rotate(270deg); /* Safari and Chrome */
    -o-transform:rotate(270deg); /* Opera */
  }
  .accordion-header{
    background-color: #f8f9fa;    
    padding-top: .5em;
    padding-bottom: .5em;
  }
  .btn{ 
    font-size: 10pt;
    height: 2.5em;
    padding-top: .3em;
  }
  .btn-start{
    margin-left: 1em;
    margin-bottom: 1em;
  }
  .preresult{
    margin-top: 1em;
    margin-bottom: 2em;
  }
  .mobile-header{
    margin-bottom: 1em;
  }
  .mobile-navigation{
    border-bottom:0px solid red;
  }
  .answer-input{
    margin-bottom: 0px;
  }
  .answer-buttons{
    margin-top: 0.9em;
  }
  .weight-active{
    margin-bottom: 2em;
  }
</style>
