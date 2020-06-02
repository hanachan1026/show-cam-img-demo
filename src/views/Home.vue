<template>
  <section class="section">
    <div class="container">
      <h1 class="title">
        Hello
        {{userName}}
      </h1>
      <p>Change camera images every 3 seconds <br>(For Demo purpose)</p>

      <select name="showCamera" v-on:change="handleShowCamera">
        <option value="0">ALL</option>
        <option value="camera1">camera1</option>
        <option value="camera2">camera2</option>
        <option value="camera3">camera3</option>
      </select>
      <div id="camera-images" style="margin-top: 20px">
        <div v-for="(camera, index) in cameras" :key="index">
          <template v-if="camera.isShow">
            <h3>{{ camera.name }}</h3>
            <li :key="camera.name" style="list-style: none;">
              <img :src="camera.images[showingIndex]" :alt="camera.name" />
            </li>
          </template>
        </div>
      </div>
    </div>
  </section>
</template>

<script lang="ts">
import { storage } from "../scripts/firebase";
import Vue from "vue";
import { Component } from "vue-property-decorator";

interface CameraImageDictionary {
  name: string;
  images: string[];
  isShow: boolean;
}

@Component
export default class Home extends Vue {
  cameras: CameraImageDictionary[] = [];
  imageCount = 0;
  showingIndex = 0;
  timer = 0;
  
  async created() { 
    const storageRef = storage.ref();
    const imagesRef = storageRef.child('images');
    const listResult1 = await imagesRef.listAll();

    const cameras: Promise<CameraImageDictionary>[] = await listResult1.prefixes.map(async (folderRef) => {
      const imageData: CameraImageDictionary = {
        name: folderRef.name,
        images: [],
        isShow: true
      };

      const listResult2 = await folderRef.listAll();

      listResult2.items.forEach(async (itemRef) => {
        const downloadURL = await itemRef.getDownloadURL();
        imageData.images.push(downloadURL);
      });

      return imageData;      
    });
    
    cameras.forEach((promise) => {
      promise.then((camera) => { 
        this.cameras.push(camera);
      });
    });

    this.cameras = this.cameras.sort((a, b) => (a.name > b.name) ? -1 : 1)

    this.timer = window.setInterval(() => { 
      this.showingIndex = Math.floor(Math.random() * 3);
    }, 3000)
  }

  get userName(): string {
    if (this.$store.state.user) {
      return this.$store.state.user.displayName;
    }

    return "";
  }

  handleShowCamera(event: Event) {
    const target = event.currentTarget as HTMLInputElement;
    const val = target.value;
    if (val === "0") {
      this.cameras.forEach(camera => {
        camera.isShow = true;
      });
    } else {
      this.cameras.forEach(camera => {
        if (camera.name === val) {
          camera.isShow = true;
        } else {
          camera.isShow = false;
        }
      });
    }
  }
}
</script>
