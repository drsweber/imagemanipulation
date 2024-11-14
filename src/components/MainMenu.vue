<template>
    <div class="menu">
      <!-- <div class="upload-wrapper"> -->
        <button
          class="menu-button"
          @click="handleButtonClick"
          value="input"
          title="Выбрать файл"
        >
          <label for="fileInput" class="button-label">
            <img
              class="button-image"
              src="../assets/upload.png"
              alt="Select Image"
            />
          </label>
          <input
            id="fileInput"
            class="button-input"
            type="file"
            @change="handleImageSelection"
          />
        </button>
      <!-- </div> -->
      <!-- <div class="options"> -->
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="hand"
          title="Переместить изображение"
        >
          <label
            class="button-label"
            :class="{
              'button-label-tapped': state === 'hand',
              disabled: !selectedImage,
            }"
          >
            <img class="button-image" src="../assets/hand.png" alt="Move Image" />
          </label>
        </button>
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="pipette"
          title="Пипетка"
        >
          <label
            class="button-label"
            :class="{
              'button-label-tapped': state === 'pipette',
              disabled: !selectedImage,
            }"
          >
            <img class="button-image" src="../assets/pipette.png" alt="Pipette" />
          </label>
        </button>
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="modal"
          title="Измененть размер"
        >
          <label
            class="button-label"
            :class="{
              'button-label-tapped': state === 'modal',
              disabled: !selectedImage,
            }"
          >
            <img class="button-image" src="../assets/modal.png" alt="modal" />
          </label>
        </button>
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="correct"
          title="Редактировать изображение"
        >
          <label class="button-label" :class="{ disabled: !selectedImage }">
            <img
              class="button-image"
              src="../assets/correction.png"
              alt="Correct Image"
            />
          </label>
        </button>
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="filter"
          title="Использовать фильтры"
        >
          <label class="button-label" :class="{ disabled: !selectedImage }">
            <img
              class="button-image"
              src="../assets/filtration.png"
              alt="Filter Image"
            />
          </label>
        </button>
        <button
          class="menu-button"
          @click="handleButtonClick"
          :disabled="!selectedImage"
          value="save"
          title="Сохранить изображение"
        >
          <label class="button-label" :class="{ disabled: !selectedImage }">
            <img
              class="button-image"
              src="../assets/downloads.png"
              alt="Save Image"
            />
          </label>
        </button>
      <!-- </div> -->
    </div>
  </template>
  
  <script>
  import { defineComponent } from "vue";
  
  export default defineComponent({
    name: "MainMenu",
    props: {
      state: String,
    },
    data() {
      return {
        selectedImage: "",
        imageUrl: "",
      };
    },
    methods: {
      handleImageSelection(event) {
        const target = event.target;
        const selectedFile = target.files?.[0];
  
        if (selectedFile) {
          const imageUrl = URL.createObjectURL(selectedFile);
          this.selectedImage = imageUrl;
          this.$emit("onImageSelected", imageUrl);
  
          target.value = "";
        }
      },
      handleButtonClick(event) {
        const target = event.target;
        const value = target.parentElement?.value
          ? target.parentElement.value
          : target.parentElement?.parentElement.value;
        this.$emit("changeState", value);
      },
    },
    beforeUnmount() {
      if (this.selectedImage) {
        URL.revokeObjectURL(this.selectedImage);
      }
    },
    watch: {
      state(newValue) {
        if (newValue === "pipette") {
          this.stateCursor = "pipette";
        } else this.stateCursor = null;
      },
    },
  });
  </script>
  
  <style scoped lang="scss">
  .menu {
    position: absolute;
    left: 20px;
    top: 50%;
    transform: translateY(-50%);
    width: 60px;
    height: 600px;
    padding: 5px;
    display: flex;
    flex-direction: column;
    gap: 15px;
    align-items: center;
    justify-content: center;
    background-color: #202020;
    border-radius: 30px;
}
  
  
  .button {
    &-input {
      width: 80%;
      display: none;
    }
  
    &-label {
      &-label {
      cursor: pointer;
      border: 1px;
      border-radius: 10px;
      display: block;
      transition: 0.3s;
      &:hover {
        background-color: #0000001e;
      }
      &-tapped {
        background: #1b1b1b;
      }
    }
  
    &-image {
      height: 25px;
      width: 25px;
      padding: 11px 12px;
    }
    }
  
    &-image {
      height: 25px;
      width: 25px;
      padding: 11px 12px;
    }
  }
  
  .menu-button {
    border: none;
    background: transparent;
    border-radius: 15px; 
    transition: background-color 0.3s;
  }
  
  .button-image {
    width: 24px;
    height: 24px;
  }
  
  .disabled {
    opacity: 0.3;
  }
  </style>