<template>
  <modal-inner aria-label="Insert image">
    <div class="modal__content">
      <p>Please provide a <b>URL</b> for your image.</p>
      <form-entry label="URL" error="url" style="margin: 5px 0;">
        <input slot="field" class="textfield" type="text" v-model.trim="url" @keydown.enter="resolve">
      </form-entry>
      <input class="hidden-file" id="import-markdown-file-input" type="file" @change="uploadFile">
      <label class="menu-entry button flex flex--row flex--align-center" for="import-markdown-file-input" style="padding-left: 0;">
        <div class="menu-entry__icon flex flex--column flex--center">
          <icon-upload></icon-upload>
        </div>
        <div class="flex flex--column">
          <div>本地上传</div>
          <span>请选择本地文件上传.</span>
        </div>
      </label>
      <!-- <menu-entry @click.native="openGooglePhotos(token)" v-for="token in googlePhotosTokens" :key="token.sub">
        <icon-provider slot="icon" provider-id="googlePhotos"></icon-provider>
        <div>Open from Google Photos</div>
        <span>{{token.name}}</span>
      </menu-entry> -->
      <!-- <menu-entry @click.native="addGooglePhotosAccount">
        <icon-provider slot="icon" provider-id="googlePhotos"></icon-provider>
        <span>Add Google Photos account</span>
      </menu-entry> -->
      <!-- <menu-entry>
          <icon-upload slot="icon"></icon-upload>
          <div>Publish now</div>
          <span>Update publications for File.</span>
          <input type="file" value="" id="file" @change="uploadFile">
        </menu-entry> -->
    </div>
    <div class="modal__button-bar">
      <button class="button" @click="reject()">取消</button>
      <button class="button button--resolve" @click="resolve">确定</button>
    </div>
  </modal-inner>
</template>

<script>
import modalTemplate from './common/modalTemplate';
import MenuEntry from '../menus/common/MenuEntry';
import googleHelper from '../../services/providers/helpers/googleHelper';
import store from '../../store';
import axios from 'axios';
import pagedownButtons from '../../data/pagedownButtons';

export default modalTemplate({
  components: {
    MenuEntry,
  },
  data: () => ({
    url: '',
  }),
  computed: {
    googlePhotosTokens() {
      const googleTokensBySub = store.getters['data/googleTokensBySub'];
      return Object.values(googleTokensBySub)
        .filter(token => token.isPhotos)
        .sort((token1, token2) => token1.name.localeCompare(token2.name));
    },
  },
  methods: {
    resolve(evt) {
      evt.preventDefault(); // Fixes https://github.com/benweet/stackedit/issues/1503
      if (!this.url) {
        this.setError('url');
      } else {
        const { callback } = this.config;
        this.config.resolve();
        callback(this.url);
      }
    },
    reject() {
      const { callback } = this.config;
      this.config.reject();
      callback(null);
    },
    async addGooglePhotosAccount() {
      try {
        await googleHelper.addPhotosAccount();
      } catch (e) { /* cancel */ }
    },
    async openGooglePhotos(token) {
      const { callback } = this.config;
      this.config.reject();
      const res = await googleHelper.openPicker(token, 'img');
      if (res[0]) {
        store.dispatch('modal/open', {
          type: 'googlePhoto',
          url: res[0].url,
          callback,
        });
      }
    },
    uploadFile(e) {
      console.log('upload file ....');
      const formData = new FormData();
      const data = JSON.stringify({
        user: 'username',
        env: 'dev',
      });
      formData.append('file', e.target.files[0]);
      formData.append('data', data);
      const url = '/api/file/add';
      const config = {
        headers: { 'Content-Type': 'multipart/form-data' },
      };
      axios.post(url, formData, config).then((response) => {
        console.log(response.data);
        if(response.data.code === 0) {
            this.url = response.data.data;
        } else {
          store.dispatch('notification/error', '抱歉，文件上传失败: ' + response.data.msg );
        }
      }).catch(err => {
        store.dispatch('notification/error', '抱歉，文件上传失败！');
      }); 
    },
  },
});
</script>
