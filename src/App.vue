<template>
    <div id="app">
      <v-app>
        <v-dialog v-model="dialog3" max-width="600px" persistent>
        <v-card>
          <v-card-title>
            <span>welcome to karma academy 🗿</span>
          </v-card-title>
          <v-card-text>
            <p>this is a collaborative dating VN</p>
            <p>feel free to add new paths anywhere. the username in the corner indicates who wrote that path</p>
            <p>please write your reddit username:</p>
            <v-text-field
              v-model="my_username"
              label="your reddit username"
              outline
            ></v-text-field>
            <p>also, you can add characters and backgrounds from the buttons at the top :) have fun!</p>
          </v-card-text>
          <v-card-actions>
            <v-btn color="primary" flat :disabled="!my_username" @click="dialog3=false">let's start!</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
        <v-slide-y-transition>
        <v-container bg fill-height grid-list-md text-xs-center v-show="!loading_inital">
          <v-layout row wrap align-center>
            <v-flex xs12>
              <v-layout column>
                <h3>karma academy</h3>
                <p>a dating sim created by redditors with ❤️</p>
              </v-layout>
                <v-layout row style="position: absolute; right: 0; top: 0; z-index: 100;" class="pa-3">
                  <v-btn small flat color="white" @click="go_to_subreddit">r/karma_academy</v-btn>
                  <gh-btns-star slug="98mprice/karma-academy" show-count></gh-btns-star>
                </v-layout>
              <v-btn v-if="current_panel"
                @click="go_back"
                dark
                :disabled="previous_panel == null"
              >
                undo
              </v-btn>
              <v-btn
                @click="start_at_beginning"
                dark
              >
                start at the beginning
              </v-btn>
              <v-dialog
                v-model="dialog_character"
                width="500"
              >
                <v-btn
                  slot="activator"
                  color="pink lighten-5"
                >
                  add new character
                </v-btn>

                <v-card>
                  <v-card-title
                    class="headline"
                    primary-title
                  >
                    {{edit_existing_character ? 'edit character' : 'add new character'}}
                    <v-btn dark
                      color="red lighten-4"
                      @click="edit_existing_character = !edit_existing_character"
                      class="mb-2 mt-0"
                    >{{edit_existing_character ? 'or add new character?' : 'or edit existing character?'}}</v-btn>
                  </v-card-title>

                  <v-card-text>
                    <v-autocomplete v-if="edit_existing_character"
                      v-model="character_name"
                      :items="items"
                      :loading="isLoading"
                      :search-input.sync="search"
                      label="name"
                      placeholder="e.g. gallowboob"
                      outline
                    ></v-autocomplete>
                    <div v-else>
                      <p>please make sure that your character doesn't already exist before adding a new one 💌</p>
                      <v-text-field
                        v-model="character_name"
                        label="name"
                        placeholder="e.g. gallowboob"
                        outline
                      ></v-text-field>
                    </div>
                    <v-text-field
                      v-model="character_url"
                      label="photo url"
                      placeholder="https://..."
                      outline
                    ></v-text-field>
                  </v-card-text>

                  <v-divider></v-divider>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                      flat
                      @click="add_character"
                    >
                      <v-icon left>add</v-icon> add
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
              <v-dialog
                v-model="dialog_background"
                width="500"
              >
                <v-btn
                  slot="activator"
                  color="pink lighten-5"
                >
                  add new background
                </v-btn>

                <v-card>
                  <v-card-title
                    class="headline"
                    primary-title
                  >
                    {{edit_existing_background ? 'edit background' : 'add new background'}}
                    <v-btn dark
                      color="red lighten-4"
                      @click="edit_existing_background = !edit_existing_background"
                      class="mb-2 mt-0"
                    >{{edit_existing_background ? 'or add new background?' : 'or edit existing background?'}}</v-btn>
                  </v-card-title>

                  <v-card-text>
                    <v-autocomplete v-if="edit_existing_background"
                      v-model="background_name"
                      :items="background_items"
                      label="name"
                      placeholder="e.g. school"
                      outline
                    ></v-autocomplete>
                    <div v-else>
                      <p>please make sure that your background doesn't already exist before adding a new one 💌</p>
                      <v-text-field
                        v-model="background_name"
                        label="name"
                        placeholder="e.g. school"
                        outline
                      ></v-text-field>
                    </div>
                    <v-text-field
                      v-model="background_url"
                      label="photo url"
                      placeholder="https://..."
                      outline
                    ></v-text-field>
                  </v-card-text>

                  <v-divider></v-divider>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                      flat
                      @click="add_background"
                    >
                      <v-icon left>add</v-icon> add
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-flex>
            <v-flex xs12 lg10 offset-lg1 v-if="current_panel">
              <v-card color="pink lighten-5">
                <div style="position: relative;">
                  <v-img v-if="current_panel.background" v-blur="true" :src="get_background_url(current_panel.background)" style="border-radius: 20px 20px 0px 0px;"></v-img>
                  <v-img v-else v-blur="true" :src="require('assets/school.png')"></v-img>
                  <v-btn color="orange" dark flat class="pa-2" style="position: absolute; top: 0; left: 0;" @click="openReddit(current_panel.reddit_username)">
                    <i class="fab fa-reddit-alien"></i> &nbsp; u/{{current_panel.reddit_username}}
                  </v-btn>
                  <v-img v-if="current_panel.character" style="position: absolute; top: 0; height: 100%; width: 40%; left: 0; right: 0; margin: auto; object-fit: scale-down;" :src="get_character_url(current_panel.character)"/>
                  <h3 style="position: absolute; left: 0; bottom: 0" class="pa-2">{{current_panel.character}}</h3>
                </div>
                <v-card flat style="min-height: 100px; border-radius: 0px 0px 20px 20px; text-align: left;">
                  <p class="pa-2" v-html="sanitize(current_panel.text.replace('${player_username}', 'u/' + my_username))"></p>
                  <p class="pl-2 pr-2 red--text" v-if="current_panel.love"><b>+ ❤️ {{current_panel.love}}</b> with {{current_panel.character}}</p>
                  <v-layout row wrap>
                    <template v-for="panel in next_panels">
                      <v-btn v-if="panel.love_req && panel.love_req < your_love[panel.character]" :disabled="loading_next_options" flat @click="next_panel(panel)">{{panel.option}}</v-btn>
                      <v-btn v-else :disabled="loading_next_options" flat @click="next_panel(panel)">{{panel.option}}</v-btn>
                    </template>
                    <v-dialog
                      v-model="dialog_option"
                      width="500"
                    >
                      <v-btn
                        slot="activator"
                        color="pink lighten-3"
                        dark
                        :disabled="loading_next_options"
                      >
                        <v-icon left>add</v-icon> new path
                      </v-btn>

                      <v-card>
                        <v-card-title
                          class="headline"
                          primary-title
                        >
                          add a new path
                        </v-card-title>

                        <v-card-text>
                          <v-text-field
                            label="option"
                            placeholder="e.g. eat bread"
                            outline
                            v-model="option"
                          ></v-text-field>
                          <p>on the next page, it'll say...</p>
                          <v-textarea
                            outline
                            label="text"
                            v-model="text"
                          ></v-textarea>
                          <v-autocomplete
                            v-model="character"
                            :items="items"
                            :loading="isLoading"
                            :search-input.sync="search"
                            label="character"
                            hint="optional"
                            outline
                            persistent-hint
                          ></v-autocomplete>
                          <v-checkbox v-show="character"
                            label="should clicking this option increase love for the assigned character?"
                            color="red"
                            v-model="checkbox"
                          ></v-checkbox>
                          <div v-show="checkbox">
                            how much love should it increase by?
                            <v-rating
                              v-model="rating"
                              full-icon="fas fa-heart"
                              empty-icon="far fa-heart"
                              color="red"
                              x-large
                              background-color="black"
                              >
                            </v-rating>
                          </div>
                          <v-checkbox v-show="character"
                            label="should there be a requirement/conditional amount of love before showing this option (if statement)?"
                            color="red"
                            v-model="checkbox_love"
                          ></v-checkbox>
                          <div v-show="checkbox_love">
                            how much love should the player have for this character, minimum?
                            <v-text-field
                              label="number of hearts"
                              placeholder="e.g. 10"
                              type="number"
                              outline
                              v-model="love_req"
                            ></v-text-field>
                          </div>
                          <v-autocomplete
                            v-model="background"
                            :items="background_items"
                            label="background"
                            hint="optional"
                            outline
                            persistent-hint
                          ></v-autocomplete>
                        </v-card-text>

                        <v-card-actions>
                          <v-spacer></v-spacer>
                          <v-btn
                            flat
                            @click="add_flow"
                          >
                            <v-icon left>add</v-icon> add
                          </v-btn>
                        </v-card-actions>
                      </v-card>
                    </v-dialog>
                  </v-layout>
                  <!--<v-card-actions style="position: absolute; bottom: 0; left: 50%; -webkit-transform: translateX(-50%); transform: translateX(-50%);">
                    <v-btn flat>option 1</v-btn>
                    <v-btn flat >option 2</v-btn>
                    <v-btn
                    color="pink lighten-3"
                    >
                      <v-icon left>add</v-icon> new option
                    </v-btn>
                  </v-card-actions>-->
                </v-card>
              </v-card>
            </v-flex>
          </v-layout>
        </v-container>
        </v-slide-y-transition>
      </v-app>
    </div>
</template>

<script>
import Vue from 'vue'
import Vuetify from 'vuetify'
import vBlur from 'v-blur'
import $ from 'jquery'
import axios from 'axios'
import VueGitHubButtons from 'vue-github-buttons';
import 'vue-github-buttons/dist/vue-github-buttons.css';
Vue.use(VueGitHubButtons);

var xssFilters = require('xss-filters');

Vue.use(vBlur)

import colors from 'vuetify/es5/util/colors'
Vue.use(Vuetify, {
  theme: {
    primary: colors.red.base
  }
})

export default {
  name: 'app',
  data () {
    return {
      edit_existing_character: false,
      edit_existing_background: false,
      dialog_option: false,
      dialog_character: false,
      dialog_background: false,
      rating: 3,
      checkbox: false,
      checkbox_love: false,
      character: null,
      current_panel: null,
      next_panels: [],
      characters: [],
      backgrounds: [],
      descriptionLimit: 60,
      entries: [],
      isLoading: false,
      character_name: null,
      character_url: null,
      background_name: null,
      background_url: null,
      search: null,
      my_username: null,
      text: null,
      option: null,
      love_req: 10,
      background: null,
      loading_next_options: false,
      test: 'hi this is a test of the <b>typewriter</b>',
      loading_inital: false,
      dialog3: true,
      your_love: {},
      previous_panel: null
    }
  },
  computed: {
    fields () {
      if (!this.model) return []

      return Object.keys(this.character_name).map(key => {
        return {
          key,
          value: this.character_name[key] || 'n/a'
        }
      })
    },
    items () {
      return this.characters.map(character => {
        const name = character.name

        return name
      })
    },
    background_items () {
      return this.backgrounds.map(background => {
        const name = background.name

        return name
      })
    }
  },
  watch: {
    search (val) {
      // Items have already been loaded
      if (this.characters.length > 0) return

      // Items have already been requested
      if (this.isLoading) return

      this.isLoading = true

      // Lazily load input items
      axios.get('https://karma-academy.herokuapp.com/get_characters')
        .then(res => {
          this.characters = res.data
        })
        .catch(err => {
          console.log(err)
        })
        .finally(() => (this.isLoading = false))
    }
  },
  methods: {
    sanitize: function(str) {
      var str2 = str.replace(/<[^\w<>]*(?:[^<>"'\s]*:)?[^\w<>]*(?:\W*s\W*c\W*r\W*i\W*p\W*t|\W*f\W*o\W*r\W*m|\W*s\W*t\W*y\W*l\W*e|\W*s\W*v\W*g|\W*m\W*a\W*r\W*q\W*u\W*e\W*e|(?:\W*l\W*i\W*n\W*k|\W*o\W*b\W*j\W*e\W*c\W*t|\W*e\W*m\W*b\W*e\W*d|\W*a\W*p\W*p\W*l\W*e\W*t|\W*p\W*a\W*r\W*a\W*m|\W*i?\W*f\W*r\W*a\W*m\W*e|\W*b\W*a\W*s\W*e|\W*b\W*o\W*d\W*y|\W*m\W*e\W*t\W*a|\W*i\W*m\W*a?\W*g\W*e?|\W*v\W*i\W*d\W*e\W*o|\W*a\W*u\W*d\W*i\W*o|\W*b\W*i\W*n\W*d\W*i\W*n\W*g\W*s|\W*s\W*e\W*t|\W*i\W*s\W*i\W*n\W*d\W*e\W*x|\W*a\W*n\W*i\W*m\W*a\W*t\W*e)[^>\w])|(?:<\w[\s\S]*[\s\0\/]|['"])(?:formaction|style|background|src|lowsrc|ping|on(?:d(?:e(?:vice(?:(?:orienta|mo)tion|proximity|found|light)|livery(?:success|error)|activate)|r(?:ag(?:e(?:n(?:ter|d)|xit)|(?:gestur|leav)e|start|drop|over)?|op)|i(?:s(?:c(?:hargingtimechange|onnect(?:ing|ed))|abled)|aling)|ata(?:setc(?:omplete|hanged)|(?:availabl|chang)e|error)|urationchange|ownloading|blclick)|Moz(?:M(?:agnifyGesture(?:Update|Start)?|ouse(?:PixelScroll|Hittest))|S(?:wipeGesture(?:Update|Start|End)?|crolledAreaChanged)|(?:(?:Press)?TapGestur|BeforeResiz)e|EdgeUI(?:C(?:omplet|ancel)|Start)ed|RotateGesture(?:Update|Start)?|A(?:udioAvailable|fterPaint))|c(?:o(?:m(?:p(?:osition(?:update|start|end)|lete)|mand(?:update)?)|n(?:t(?:rolselect|extmenu)|nect(?:ing|ed))|py)|a(?:(?:llschang|ch)ed|nplay(?:through)?|rdstatechange)|h(?:(?:arging(?:time)?ch)?ange|ecking)|(?:fstate|ell)change|u(?:echange|t)|l(?:ick|ose))|m(?:o(?:z(?:pointerlock(?:change|error)|(?:orientation|time)change|fullscreen(?:change|error)|network(?:down|up)load)|use(?:(?:lea|mo)ve|o(?:ver|ut)|enter|wheel|down|up)|ve(?:start|end)?)|essage|ark)|s(?:t(?:a(?:t(?:uschanged|echange)|lled|rt)|k(?:sessione|comma)nd|op)|e(?:ek(?:complete|ing|ed)|(?:lec(?:tstar)?)?t|n(?:ding|t))|u(?:ccess|spend|bmit)|peech(?:start|end)|ound(?:start|end)|croll|how)|b(?:e(?:for(?:e(?:(?:scriptexecu|activa)te|u(?:nload|pdate)|p(?:aste|rint)|c(?:opy|ut)|editfocus)|deactivate)|gin(?:Event)?)|oun(?:dary|ce)|l(?:ocked|ur)|roadcast|usy)|a(?:n(?:imation(?:iteration|start|end)|tennastatechange)|fter(?:(?:scriptexecu|upda)te|print)|udio(?:process|start|end)|d(?:apteradded|dtrack)|ctivate|lerting|bort)|DOM(?:Node(?:Inserted(?:IntoDocument)?|Removed(?:FromDocument)?)|(?:CharacterData|Subtree)Modified|A(?:ttrModified|ctivate)|Focus(?:Out|In)|MouseScroll)|r(?:e(?:s(?:u(?:m(?:ing|e)|lt)|ize|et)|adystatechange|pea(?:tEven)?t|movetrack|trieving|ceived)|ow(?:s(?:inserted|delete)|e(?:nter|xit))|atechange)|p(?:op(?:up(?:hid(?:den|ing)|show(?:ing|n))|state)|a(?:ge(?:hide|show)|(?:st|us)e|int)|ro(?:pertychange|gress)|lay(?:ing)?)|t(?:ouch(?:(?:lea|mo)ve|en(?:ter|d)|cancel|start)|ime(?:update|out)|ransitionend|ext)|u(?:s(?:erproximity|sdreceived)|p(?:gradeneeded|dateready)|n(?:derflow|load))|f(?:o(?:rm(?:change|input)|cus(?:out|in)?)|i(?:lterchange|nish)|ailed)|l(?:o(?:ad(?:e(?:d(?:meta)?data|nd)|start)?|secapture)|evelchange|y)|g(?:amepad(?:(?:dis)?connected|button(?:down|up)|axismove)|et)|e(?:n(?:d(?:Event|ed)?|abled|ter)|rror(?:update)?|mptied|xit)|i(?:cc(?:cardlockerror|infochange)|n(?:coming|valid|put))|o(?:(?:(?:ff|n)lin|bsolet)e|verflow(?:changed)?|pen)|SVG(?:(?:Unl|L)oad|Resize|Scroll|Abort|Error|Zoom)|h(?:e(?:adphoneschange|l[dp])|ashchange|olding)|v(?:o(?:lum|ic)e|ersion)change|w(?:a(?:it|rn)ing|heel)|key(?:press|down|up)|(?:AppComman|Loa)d|no(?:update|match)|Request|zoom))[\s\0]*=/gi, " ")
      return str2
    },
    openReddit: function(username) {
      window.open('http://www.reddit.com/u/' + username)
    },
    go_to_subreddit: function() {
      window.open('http://www.reddit.com/r/karma_academy')
    },
    get_character_url: function(character_name) {
      for (let character of this.characters) {
        if (character.name == character_name) {
          return character.url
        }
      }
    },
    get_background_url: function(background_name) {
      for (let background of this.backgrounds) {
        if (background.name == background_name) {
          return background.url
        }
      }
    },
    next_panel: function(panel) {
      this.previous_panel = this.current_panel
      this.current_panel = panel
      this.loading_next_options = true
      if (panel.love) {
        if (!this.your_love[panel.character]) {
          this.your_love[panel.character] = 0
        }
        this.your_love[panel.character] += panel.love
      }
      let vm = this
      $.ajax({
        "async": true,
        "crossDomain": true,
        "url": "https://karma-academy.herokuapp.com/get_flows",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json",
          "cache-control": "no-cache",
          "Postman-Token": "ada39ab1-6a6c-4f38-b672-8ade1d3cda1f"
        },
        "processData": false,
        "data": JSON.stringify({ id: panel._id['$oid'] })
      }).done(function (response) {
        vm.next_panels = response
        vm.loading_next_options = false
      });
    },
    add_character: function() {
      var settings = {
        "async": true,
        "crossDomain": true,
        "url": "https://karma-academy.herokuapp.com/add_character",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json",
          "cache-control": "no-cache",
          "Postman-Token": "6f60a85d-d422-49d8-9028-f388e119dc5b"
        },
        "processData": false,
        "data": JSON.stringify({
          name: this.character_name,
          url: this.character_url
        })
      }

      this.dialog_character = false
      let vm = this
      $.ajax(settings).done(function (response) {
        vm.characters.push({
          name: this.character_name,
          url: this.character_url
        })
      });
    },
    add_background: function() {
      var settings = {
        "async": true,
        "crossDomain": true,
        "url": "https://karma-academy.herokuapp.com/add_background",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json",
          "cache-control": "no-cache",
          "Postman-Token": "6f60a85d-d422-49d8-9028-f388e119dc5b"
        },
        "processData": false,
        "data": JSON.stringify({
          name: this.background_name,
          url: this.background_url
        })
      }

      this.dialog_background = false
      let vm = this
      $.ajax(settings).done(function (response) {
        vm.backgrounds.push({
          name: this.background_name,
          url: this.background_url
        })
      });
    },
    add_flow: function() {
      let data = {
        option: this.option,
        text: this.text,
        character: this.character,
        background: this.background,
        previous_id: this.current_panel._id['$oid'],
        reddit_username: this.my_username
      }
      if (this.checkbox) {
        data.love = this.rating
      }
      if (this.checkbox_love) {
        data.love_req = this.love_req
      }
      var settings = {
        "async": true,
        "crossDomain": true,
        "url": "https://karma-academy.herokuapp.com/add_flow",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json",
          "cache-control": "no-cache",
          "Postman-Token": "6f60a85d-d422-49d8-9028-f388e119dc5b"
        },
        "processData": false,
        "data": JSON.stringify(data)
      }

      this.dialog_option = false
      let vm = this
      $.ajax(settings).done(function (response) {
        vm.next_panels.push(response)
        vm.text = '',
        vm.option = ''
        vm.character = ''
        vm.love = 3
        vm.love_req = 10
      });
    },
    start_at_beginning: function() {
      this.loading_inital = true
      let vm = this
      $.ajax({
        "async": true,
        "crossDomain": true,
        "url": "https://karma-academy.herokuapp.com/get_flows",
        "method": "POST",
        "headers": {
          "Content-Type": "application/json",
          "cache-control": "no-cache",
          "Postman-Token": "ada39ab1-6a6c-4f38-b672-8ade1d3cda1f"
        },
        "processData": false,
        "data": "{\n\t\"id\": \"-1\"\n}"
      }).done(function (response) {
        vm.current_panel = response[0]
        let current_id = response[0]._id['$oid']
        $.ajax({
          "async": true,
          "crossDomain": true,
          "url": "https://karma-academy.herokuapp.com/get_flows",
          "method": "POST",
          "headers": {
            "Content-Type": "application/json",
            "cache-control": "no-cache",
            "Postman-Token": "ada39ab1-6a6c-4f38-b672-8ade1d3cda1f"
          },
          "processData": false,
          "data": JSON.stringify({ id: current_id })
        }).done(function (response) {
          vm.next_panels = response
          vm.loading_inital = false
        });
      });
    },
    go_back: function(){
      this.next_panel(this.previous_panel)
    }
  },
  mounted: function() {

    this.start_at_beginning()

    axios.get('https://karma-academy.herokuapp.com/get_characters')
      .then(res => {
        this.characters = res.data
      })
      .catch(err => {
        console.log(err)
      })

      axios.get('https://karma-academy.herokuapp.com/get_backgrounds')
        .then(res => {
          this.backgrounds = res.data
        })
        .catch(err => {
          console.log(err)
        })

    /*var text = $('.typewriter').text();

    var length = text.length;
    var timeOut;
    var character = 0;

    (function typeWriter() {
        timeOut = setTimeout(function() {
            character++;
            var type = text.substring(0, character);
            $('.typewriter').text(type);
            typeWriter();

            if (character == length) {
                clearTimeout(timeOut);
            }

        }, 15);
    }());*/
  }
}
</script>

<style>
.application {
  background: transparent !important;
}
body {
  background-color: pink;
}
.v-card {
  border-radius: 20px;
  box-shadow: 10px 10px rgba(255, 255, 255, 0.3);
}
.v-btn {
  border-radius: 20px;
  text-transform: lowercase;
  box-shadow: 0px 0px !important;
}
.v-dialog {
  border-radius: 20px;
  box-shadow: 10px 10px rgba(255, 255, 255, 0.3);
}
.v-select-list {
  border-radius: 4px;
}
.v-image__image {
  background-position: center top !important;
}
</style>
