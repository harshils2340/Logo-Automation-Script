<template>
    <div id="app">
      <h1>{{ currentSchool }}</h1>
      <div class="logo-container">
        <img v-for="logo in logoOptions" :key="logo" :src="logo" :alt="currentSchool + ' logo'"/>
      </div>
      <div class="buttons">
        <button id="approve-btn" @click="approve">Approve</button>
        <button id="deny-btn" @click="deny">Deny</button>
      </div>
      <div class="message" v-if="message">{{ message }}</div>
    </div>
  </template>
  
    <script>
      const axios = require("axios");
      const cheerio = require("cheerio");
      const fs = require("fs");
  
      new Vue({
        el: "#app",
        data() {
          return {
            schoolWithLogos: {},
            schools: [
              "Harvard University",
              "Princeton University",
              "University of Waterloo",
              "University of Toronto"
            ],
            currentSchool: '',
            currentLogo: '',
            approved: false,
            denied: false,
            message: '',
            messageColor: '',
            messageDelay: 3000
          }
        },
        methods: {
          async getLogoOptions(school) {
            try {
                const searchQuery = `${school.replace(/\s+/g, "+")}+logo`; // replaces spaces in school with + to match google search query
                const searchUrl = `https://www.google.com/search?q=${searchQuery}`;
                const response = await axios.get(searchUrl); // get request to the school name logo
                const data = cheerio.load(response.data);
  
                let imgUrlArr = ["https://upload.wikimedia.org/wikipedia/commons/thumb/7/70/Harvard_University_logo.svg/800px-Harvard_University_logo.svg.png"];
                const imgOptions = 5;
                let getImgUrl = ($) => { // callback function to get the first 5 images to check for logo incase first ones arent correct
                  for (let i = 0; i < imgOptions; i++) {
                    let img = $.getElementsByClassName("rg_i")[i].getAttribute("src");
                    imgUrlArr.push(img);
                  }
                }
                getImgUrl(data);
                this.logoOptions = imgUrlArr;
              }
  
              catch (error) {
              console.error('Error occurred while downloading school logos:', error);
              console.log("Sorry, no valid images were found.")
            }
          },
          
          async downloadSchoolLogos(schools) {
            try {
              for (let school of schools) {
                this.currentSchool = school;
                await this.getLogoOptions(school);
  
                for (let img of this.logoOptions) {
                  this.currentLogo = img;
                  if (this.approved) {
                    let schoolName = `${school.replace(/\s+/g)}`;
                    let logo = `${school}-logo.png`;
                    let logoPath = `schoolLogos/${logo}`;
  
                    if (!this.schoolWithLogos.hasOwnProperty(schoolName)) {
                      this.schoolWithLogos[schoolName] = { logo: logoPath };
                    }
                    else {
                      this.schoolWithLogos[schoolName].logo = logoPath;
                    }
                    this.setMessage("Logo Saved Successfully", "green");
                    this.approved = false;
                    break;
                  }
                  if (this.denied) {
                    this.setMessage("Logo Denied Successfully", "red");
                    this.denied = false;
                  }
                }
              }
              fs.writeFileSync("schoolLogos.json", JSON.stringify(this.schoolWithLogos)); // write school logos data to a json file
  
            }
  
            catch (error) {
              console.error('Error occurred while downloading school logos:', error);
              console.log("Sorry, no valid images were found.")
            }
          },
          setMessage(msg, color) {
            this.message = msg;
            this.messageColor = color;
            setTimeout(() => {
              this.message = '';
              this.messageColor = '';
            }, messageDelay);
          }, 
           // async downloadLogos() {
           // const emptySchoolLogoNames = await db.schools('schools').find({ "schoolLogo": '' }).toArray();
           // this.schools = emptySchoolLogoNames.map(({ locations }) => locations.map(({ name }) => name)).flat();
           // await this.downloadSchoolLogos(this.schools);
           //},
          approve() {
            this.approved = true;
            this.denied = false;
          },
          deny() {
            this.approved = false;
            this.denied = true;
          },
        },
      })
    </script>
  
    <style>
      body {
        margin: 0;
        padding: 0;
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
      }
  
      #app {
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        text-align: center;
        border: 1px solid black;
        padding: 100px;
      }
  
      h1 {
        font-size: 2rem;
        margin-top: 0;
      }
  
      img {
        width: 100px;
        height: auto;
        margin: 20px 0;
      }
  
      .buttons {
        display: flex;
        justify-content: center;
        align-items: center;
        margin-top: 20px;
      }
  
      #approvebtn {
        background-color: green;
        color: white;
        padding: 10px;
        border-radius: 5px;
        margin-right: 10px;
      }
  
      #denybtn {
        background-color: red;
        color: white;
        padding: 10px;
        border-radius: 5px;
        margin-left: 10px;
      }
  
      #approved {
        color: green;
        margin-top: 20px;
      }
  
      #denied {
        color: red;
        margin-top: 20px;
      }
    </style>
