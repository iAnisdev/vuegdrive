<template>
  <div class="home">
    <input v-model="title" placeholder="Enter you title" />
    <br />
    <br />
    <textarea  v-model="textarea" cols="16" rows="10" placeholder="Enter you file content" ></textarea>
    <br />
    <button @click="addtoDrive">Submit</button>
    <br />
    <p>Drive API Quickstart</p>

    <button id="authorize_button"  @click="handleAuthClick">Authorize</button>

    <pre id="content" style="white-space: pre-wrap;"></pre>
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Home',
  data() {
    return {
      title: '',
      textarea: ''
    }
  },
  methods:{
    handleClientLoad() {
        gapi.load('client:auth2', this.initClient);
      },
     async initClient() {
        let _self = this
        await gapi.client.init({
          apiKey: API_KEY,
          clientId: CLIENT_ID,
          discoveryDocs: DISCOVERY_DOCS,
          scope: SCOPES
        }).then(function (resp) {
          console.log("resp " , resp)
          gapi.auth2.getAuthInstance().isSignedIn.listen(_self.updateSigninStatus);
          _self.updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
        }, function(error) {
          console.log("error ", error)
        });
      },
      updateSigninStatus(isSignedIn) {
        if (isSignedIn) {
          this.isSignedIn =true
        } else {
          this.isSignedIn =true
        }
      },
      handleAuthClick(event) {
        gapi.auth2.getAuthInstance().signIn();
      },
      handleSignoutClick(event) {
        gapi.auth2.getAuthInstance().signOut();
      },
      async addtoDrive(){
        console.log("gapi.client " , gapi.client)
       await gapi.client.request({
          'path': '/drive/v3/files',
          'method': 'POST',
          'body': {
            'name' : `${this.title}.txt`
          }
          }).then((request) =>{
            console.log("request.result.id, " ,request)
            return gapi.client.request({
          path: '/upload/drive/v3/files/' + request.result.id,
          method: 'PATCH',
          params: {
            uploadType: 'media'
          },
          body: 'testing file content'
        })
          })
         
    }
  },
  mounted(){
    setTimeout(() => {
      this.handleClientLoad()
    }, 1000);
  }
}
</script>
