<template>
  <div>
    <div class="text-center">
      <v-overlay :value="isLoading">
        <v-progress-circular indeterminate size="64"></v-progress-circular>
      </v-overlay>
    </div>

    <!--筐体接続カード-->
    <v-card v-if="!isSetServerIP" class="ma-4">
      <v-toolbar color="primary" dark flat>
        <v-toolbar-title>筐体接続</v-toolbar-title>
      </v-toolbar>
      <form>
        <v-text-field
            v-model="ServerIP"
            class="mx-4 pt-5"
            label="IPアドレス"
        ></v-text-field>
        <v-layout justify-end>
            <v-btn class="ma-4" @click="Connect" color="primary">Connect</v-btn>
        </v-layout>
      </form>
    </v-card>

    <!--ゲーム諸注意カード-->
    <v-card v-if="!isGaming&&isSetServerIP" class="ma-4">
      <v-toolbar color="red" dark flat>
        <v-toolbar-title>ゲームをプレイするにあたって</v-toolbar-title>
      </v-toolbar>
      <v-card-text>このゲームはプレイ中に動き回ります、けがをしないようにお気を付けください。</v-card-text>
    </v-card>

    <!--あそびかた
    <v-card v-if="!isGaming&&isSetServerIP" class="ma-4">
      <v-toolbar color="primary" dark flat>
        <v-toolbar-title>あそびかた</v-toolbar-title>
      </v-toolbar>
      <v-card-text>電子部品</v-card-text>
    </v-card>-->

    <!--キャラクター紹介カード
    <div v-if="!isGaming&&!isSetServerIP&&!isSetNextGame">
      <v-card class="ma-4">
        <v-toolbar color="primary" dark flat>
          <v-toolbar-title>キャラクター紹介</v-toolbar-title>
        </v-toolbar>
        <v-window v-model="onboarding" vertical>
          <v-window-item v-for="n in list">
          <v-img
            :src="n.ing"
            aspect-ratio="1"
            class="grey lighten-2 ma-1"
            max-height="500"
          ></v-img>
            <h1 style="font-size: 3rem" class="mx-4">{{ n.title }}</h1>
            <div style="font-size: 1rem" class="mx-4">{{ n.descliption }}</div>
          </v-window-item>
        </v-window>
        <v-card-actions class="justify-space-between">
          <v-btn
            text
            @click="prev"
          >
            <v-icon>mdi-chevron-left</v-icon>
          </v-btn>
          <v-item-group
            v-model="onboarding"
            class="text-center"
            mandatory
          >
            <v-item
              v-for="n in length"
              :key="`btn-${n}`"
              v-slot:default="{ active, toggle }"
            >
              <v-btn
                :input-value="active"
                icon
                @click="toggle"
              >
                <v-icon>mdi-record</v-icon>
              </v-btn>
            </v-item>
          </v-item-group>
          <v-btn
            text
            @click="next"
          >
            <v-icon>mdi-chevron-right</v-icon>
          </v-btn>
        </v-card-actions>
      </v-card>
    </div>-->

    <!--ゲームエントリーカード-->
    <v-card v-if="!isGaming&&isSetServerIP&&!isSetNextGame" class="ma-4">
      <v-toolbar color="primary" dark flat>
        <v-toolbar-title>ゲームエントリー</v-toolbar-title>
      </v-toolbar>
      <form>
        <v-select
          :items="ModeList"
          item-text="displayName"
          item-value="queryName"
          class="mx-4 pt-4"
          label="ゲームモード"
          v-model="GameMode"
          persistent-hint>
        </v-select>
        <div class="mx-4 pt-4">プレイヤー1</div>
        <v-text-field
            v-model="P1Name"
            class="mx-4"
            label="なまえ"
            :counter="10"
            :rules="nameRules"
            required
        ></v-text-field>
        <v-select
          :items="CampList"
          item-text="displayName"
          item-value="queryName"
          class="mx-4"
          label="キャラクター"
          v-model="P1Camp"
          persistent-hint>
        </v-select>
        <div class="mx-4 pt-4">プレイヤー2</div>
        <v-text-field
            v-model="P2Name"
            class="mx-4"
            label="なまえ"
            :counter="10"
            :rules="nameRules"
            required
        ></v-text-field>
        <v-select
          :items="CampList"
          item-text="displayName"
          item-value="queryName"
          class="mx-4"
          label="キャラクター"
          v-model="P2Camp"
          persistent-hint>
        </v-select>
        <v-layout justify-end>
            <v-btn class="ma-4" @click="Entry" color="primary">Entry</v-btn>
        </v-layout>
      </form>
    </v-card>

    <!--落下ボタン-->
    <div class="text-center">
      <v-btn
        width="90%"
        height="500"
        class="ma-4"
        color="primary"
        v-if="isGaming"
        @click="FallObject"
        >
        <div class="display-4">おとす</div>
      </v-btn>
    </div>

  </div>
</template>

<script>
import Cookies from "js-cookie";
import axios from "axios";

export default {
  name: "App",
  components: {
  },
  data() {
    return {
      isLoading: false,
      isNeedSetNextGame: false,
      ServerIP: "",
      P1Name: "",
      P2Name: "",
      P1Camp: "",
      P2Camp: "",
      GameMode: "",
      isGaming: false,
      isSetNextGame: false,
      isSetServerIP: false,
      nameRules: [
        v => !!v || '入力必須項目です!',
        v => v.length <= 10 || '10文字以下でおなしゃす!',
      ],
      CampList: [
        { displayName: "抵抗君", queryName: "Teikou"},
        { displayName: "コンデンサ君", queryName: "Kondensa"},
        { displayName: "コイル君", queryName: "Koiru"}
      ],
      ModeList: [
        { displayName: "VSバトル", queryName: "VS"},
        { displayName: "タワーバトル", queryName: "TOWER"},
        /*{ displayName: "みんなでわいわい", queryName: "WaiWai"}*/
      ],
      list: [
        {title: "抵抗君", img: "/img/Teikou", descliption: "体重: 20kg, 電気を流れにくくするために使うすごいやつ"},
        {title: "コンデンサ君", img: "/img/Kondensa", descliption: "体重: 20kg, 電気を蓄えることができるすごいやつ"},
        {title: "コイル君", img: "/img/Koiru", descliption: "体重: 30kg, 電気と時期を互いに作用させるすごいやつ"},
      ],
      length: 3,
      onboarding: 0,
    };
  },
  created: function() {
  },
  computed: {
  },
  methods: {
    next () {
      this.onboarding = this.onboarding + 1 === this.length
        ? 0
        : this.onboarding + 1
    },
    prev () {
      this.onboarding = this.onboarding - 1 < 0
        ? this.length - 1
        : this.onboarding - 1
    },
    Connect: function(){
      //console.log(this.ServerIP);
      this.isLoading = true;
      let url = "http://" + this.ServerIP + ":9000/200.OK";
      let self = this;
      axios
        .get(url, { timeout : 5000 })
          .then(function(response){
            self.isSetServerIP = true;
            self.fetchGameState();
            setInterval(function() {self.fetchGameState();}, 5000);
          })
          .catch(error => {
            window.alert(error)
        });
      this.isLoading = false;
    },
    FallObject: function(){
      this.isLoading = true;
      let url = "http://" + this.ServerIP + ":9000/FallObject";
      let self = this;
      axios
        .get(url, { timeout : 5000 })
          .then(function(response){
          })
          .catch(error => {
            window.alert(error)
        });
      this.isLoading = false;
    },
    fetchGameState: function(){
      let self = this;
      var url = "http://" + this.ServerIP + ":9000/isGaming";
      axios
        .get(url, { timeout : 5000 })
          .then(function(response){
            if(response.data == true){
              self.isGaming = true;
              console.log("Gaming");
            }
            if(response.data == false){
              self.isGaming = false;
            }
          })
          .catch(error => {
            self.isSetServerIP = false;
            window.alert(error)
        });
      url = "http://" + this.ServerIP + ":9000/isSetNextGame";
      axios
        .get(url, { timeout : 5000 })
          .then(function(response){
            if(response.data == true){
              self.isSetNextGame = true;
            }
            if(response.data == false){
              self.isSetNextGame = false;
            }
          })
          .catch(error => {
            self.isSetServerIP = false;
            window.alert(error);
        });
    },
    Entry: function(){
      this.isLoading = true;
      let url = "http://" + this.ServerIP + ":9000/SetNextGame";
      let self = this;
      let QueryString = "";
      if(this.GameMode=="VS"||this.GameMode=="TOWER"){
        if(this.P1Camp==""||this.P2Camp==""||this.P1Name==""||this.P2name==""){
          window.alert("入力していない項目があります");
          this.isLoading = false;
          return 0;
        }
        if(this.P1Name.indexOf('?')!=-1||this.P1Name.indexOf('&')!=-1||this.P1Name.indexOf('=')!=-1){
          window.alert("プレイヤー1の名前に使用できない文字があります");
          this.isLoading = false;
          return 0;
        }
        if(this.P2Name.indexOf('?')!=-1||this.P2Name.indexOf('&')!=-1||this.P2Name.indexOf('=')!=-1){
          window.alert("プレイヤー2の名前に使用できない文字があります");
          this.isLoading = false;
          return 0;
        }
        QueryString += "?GameMode="+this.GameMode;
        QueryString += "&P1Name=" + this.P1Name;
        QueryString += "&P2Name=" + this.P2Name;
        QueryString += "&P1Camp=" + this.P1Camp;
        QueryString += "&P2Camp=" + this.P2Camp;
      }
      else{
        window.alert("モードが不正");
        this.isLoading = false;
        return 0;
      }
      console.log(url+QueryString);
      axios
        .get(url+QueryString, { timeout : 5000 })
          .then(function(response){
            self.isSetNextGame = true;
          })
          .catch(error => {
            window.alert(error)
        });
      this.isLoading = false;
    }
  }
};
</script>
