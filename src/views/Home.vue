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
      var CLIENT_ID = '';
      var CLIENT_SECRET = '';
      var API_KEY = ''
      var DISCOVERY_DOCS = ["https://www.googleapis.com/discovery/v1/apis/drive/v3/rest"];
      var SCOPES = 'https://www.googleapis.com/auth/drive.file';
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
          // Listen for sign-in state changes.
          gapi.auth2.getAuthInstance().isSignedIn.listen(_self.updateSigninStatus);
// console.log("gapi.auth2.getAuthInstance().isSignedIn.get() " , gapi.auth2.getAuthInstance().isSignedIn.get())
          // Handle the initial sign-in state.
          _self.updateSigninStatus(gapi.auth2.getAuthInstance().isSignedIn.get());
        }, function(error) {
          _self.appendPre(JSON.stringify(error, null, 2));
        });
      },
      updateSigninStatus(isSignedIn) {
        if (isSignedIn) {
          this.listFiles();
        } else {
        }
      },
      handleAuthClick(event) {
        gapi.auth2.getAuthInstance().signIn();
      },
      handleSignoutClick(event) {
        gapi.auth2.getAuthInstance().signOut();
      },
      appendPre(message) {
        var pre = document.getElementById('content');
        var textContent = document.createTextNode(message + '\n');
        pre.appendChild(textContent);
      },
      listFiles() {
        let _self = this
        gapi.client.drive.files.list({
          'pageSize': 10,
          'fields': "nextPageToken, files(id, name)"
        }).then(function(response) {
          _self.appendPre('Files:');
          var files = response.result.files;
          if (files && files.length > 0) {
            for (var i = 0; i < files.length; i++) {
              var file = files[i];
              _self.appendPre(file.name + ' (' + file.id + ')');
            }
          } else {
            _self.appendPre('No files found.');
          }
        })
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
