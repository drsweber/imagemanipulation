<template>
    <div class="editor">
      <MainCanvas
        ref="canvas"
        :state="state"
        :img="img"
        :origImg="origImg"
        :newImg="newImg"
        :scale="scale"
        :interpolationCb="interpolationCb"
        :newiw="newiw"
        :newih="newih"
        :isShowCorrection="isShowCorrection"
        :isShowFiltration="isShowFiltration"
        @updateImageSizes="handleUpdateImageSizes"
        @updateColor="updateColor"
        @updateCoordinates="updateCoordinates"
        @closeCorrection="closeCorrection"
        @closeFiltration="closeFiltration"
        @revertNewImg="revertNewImg"
        @updateNewImgData="updateNewImgData"
      />
      <StatusBar
        :state="state"
        :imageHeight="ih"
        :imageWidth="iw"
        :pickedColor="pickedColor"
        :xMouse="xMouse"
        :yMouse="yMouse"
        :scale="scale"
        :canvasWidth="originalWidth"
        :canvasHeight="originalHeight"
        :imgLoaded="imgLoaded"
        @updateScale="(value) => (scale = value)"
      />
    </div>
    <ModalWindow v-show="isShowModal" @close="closeModal">
      <template v-slot:body>
        <div class="modal">
          <div class="measurements-wrapper">
            <label for="resizeUnit">Единицы измерения:</label>
            <select
              class="resize-unit"
              id="resizeUnit"
              v-model="resizeUnit"
              @change="updateModal($event)"
            >
              <option value="percentage">Проценты</option>
              <option value="pixels">Пиксели</option>
            </select>
          </div>
          <div class="interpolation">
            <label for="interpolation">Интерполяция:</label>
            <select v-model="interpolation" id="interpolation">
              <option>Нет</option>
              <option
                value="nearestNeighbor"
                title="Ближайший сосед: каждому пикселю в новом изображении присваивается значение ближайшего пикселя в исходном изображении."
              >
                Ближайших соседей
              </option>
            </select>
          </div>
          <div class="sizes">
            <div class="input-with-prefix">
              <label for="width" :style="{ left: '3px' }">W</label>
              <label for="width" class="type" :style="{ right: '3px' }">
                {{ resizeUnit === "pixels" ? "px" : "%" }}
              </label>
              <input id="width" v-model.number="newiw" @change="updateNewiw" />
            </div>
            <div class="custom-checkbox">
              <input type="checkbox" v-model="bind" />
              <img
                v-if="bind"
                class="checkbox-image"
                src="../assets/unlock.png"
                alt="Unlock"
              />
              <img
                v-else
                class="checkbox-image"
                src="../assets/lock.png"
                alt="Lock"
              />
            </div>
            <div class="input-with-prefix">
              <label for="height" :style="{ left: '6px' }"> H</label>
              <label for="height" class="type" :style="{ right: '3px' }">
                {{ resizeUnit === "pixels" ? "px" : "%" }}
              </label>
              <input id="height" v-model.number="newih" @change="updateNewih" />
            </div>
          </div>
          <div class="sizes-counter">
            <p>До: {{ getResolutionNow() }} MPs</p>
            <p>После: {{ getResolutionAfter() }} MPs</p>
          </div>
        </div>
      </template>
      <template v-slot:footer>
        <button @click="handleConfirm" class="confirm-button">Подтвердить</button>
      </template>
    </ModalWindow>
    <SideBar
      v-show="isShowPanel"
      :state="state"
      :pickedColor="pickedColor"
      :xMouse="xMouse"
      :yMouse="yMouse"
      @close="closePanel"
      @changeState="(value) => $emit('changeState', value)"
    />
  </template>
  
  <script>
  import { defineComponent } from "vue";
  import { nearestNeighborInterpolation } from "../interpolations.js";
  
  import ModalWindow from "./ModalWindow.vue";
  import StatusBar from "./StatusBar.vue";
  import MainCanvas from "./MainCanvas.vue";
  import SideBar from "./SideBar.vue";
  
  export default defineComponent({
    name: "MainEditor",
    components: { ModalWindow, StatusBar, MainCanvas, SideBar },
    props: {
      img: String,
      state: String,
    },
    data() {
      return {
        canvasRef: null,
        newImg: null,
        origImg: null,
        isShowPanel: false,
        isShowCorrection: false,
        isShowFiltration: false,
  
        resizeUnit: "pixels",
        iw: null,
        ih: null,
        newiw: null,
        newih: null,
        bind: false,
        interpolation: "",
        isShowModal: false,
  
        xMouse: null,
        yMouse: null,
        pickedColor: "",
        scale: 100,
        originalWidth: 0,
        originalHeight: 0,
        imgLoaded: false,
      };
    },
    emits: ["changeState"],
    mounted() {
      this.canvasRef = this.$refs.canvas ? this.$refs.canvas.getCanvasRef() : null;
      if (this.canvasRef) {
        this.updateImageSizes(this.canvasRef.width, this.canvasRef.height);
      }
    },
    methods: {
      closeModal() {
        this.isShowModal = false;
        this.$emit("changeState", "");
      },
      closePanel() {
        this.isShowPanel = false;
        this.$emit("changeState", "");
      },
      closeCorrection() {
        this.isShowCorrection = false;
        this.$emit("changeState", "");
      },
      closeFiltration() {
        this.isShowFiltration = false;
        this.$emit("changeState", "");
      },
      updateImageSizes(iw, ih) {
        this.iw = iw;
        this.ih = ih;
        this.imgLoaded = true; // Устанавливаем флаг загрузки изображения
      },
      updateColor(color) {
        this.pickedColor = color;
      },
      updateCoordinates([x, y]) {
        this.xMouse = x;
        this.yMouse = y;
      },
      handleConfirm() {
        this.scale = 100;
        // force update
        const image = new Image();
        image.src = this.img;
  
        if (this.resizeUnit === "pixels") {
          image.width = this.newiw;
          image.height = this.newih;
        }
        if (this.resizeUnit === "percentage") {
          const { width, height } = this.origImg; // изменяем canvasRef на origImg
          const widthPercent = width / 100;
          const heightPercent = height / 100;
          image.width = this.newiw * widthPercent;
          image.height = this.newih * heightPercent;
        }
        this.iw = image.width;
        this.ih = image.height;
        this.newImg = image;
        this.isShowModal = false;
        this.$emit("changeState", "");
      },
      updateModal() {
        if (!this.canvasRef) {
          return;
        }
        if (this.resizeUnit === "pixels") {
          if (this.newiw === null) {
            this.newiw = this.iw;
            if (this.canvasRef.width < this.iw) {
              this.newiw = this.canvasRef.width;
            }
          }
  
          if (this.newih === null) {
            this.newih = this.ih;
            if (this.canvasRef.height < this.ih) {
              this.newih = this.canvasRef.height;
            }
          }
        }
        if (this.resizeUnit === "percentage") {
          const { width, height } = this.canvasRef;
          const widthPercent = width / 100;
          const heightPercent = height / 100;
  
          if (this.newiw == null) {
            this.newiw = ~~(this.iw / widthPercent);
            this.newih = ~~(this.ih / heightPercent);
          }
        }
      },
      updateNewiw(event) {
        const num = +event.target.value;
  
        if (!isNaN(num) && num > 0) {
          this.newiw = num;
  
          if (this.canvasRef && this.canvasRef.width < num) {
            this.newiw = this.canvasRef.width;
          }
  
          if (this.bind) {
            const coef = this.ih / this.iw;
            this.newih = ~~(num * coef);
          }
        } else {
          this.newiw += 1;
          this.newiw -= 1;
        }
      },
      updateNewih(event) {
        const num = +event.target.value;
  
        if (!isNaN(num) && num > 0) {
          this.newih = num;
  
          if (this.canvasRef && this.canvasRef.height < num) {
            this.newih = this.canvasRef.height;
          }
  
          if (this.bind) {
            const coef = this.iw / this.ih;
            this.newiw = ~~(num * coef);
          }
        } else {
          this.newih += 1;
          this.newih -= 1;
        }
      },
      getResolutionNow() {
        return ((this.iw * this.ih) / 1000000).toFixed(2);
      },
      getResolutionAfter() {
        if (this.resizeUnit === "pixels") {
          return ((this.newiw * this.newih) / 1000000).toFixed(2);
        }
        if (this.resizeUnit === "percentage") {
          const { width, height } = this.canvasRef;
          const percentage = (width / 100) * (height / 100);
  
          return ((this.newiw * this.newih * percentage) / 1000000).toFixed(2);
        }
      },
      interpolationCb(img, iw, ih) {
        if (this.interpolation === "nearestNeighbor") {
          return nearestNeighborInterpolation(img, iw, ih);
        }
        return null;
      },
      closeModals() {
        this.isShowPanel = false;
        this.isShowModal = false;
        this.isShowCorrection = false;
        this.isShowFiltration = false;
      },
      revertNewImg() {
        const clear = new Image();
        clear.width = this.newImg.width;
        clear.height = this.newImg.height;
        clear.src = this.newImg.src;
        this.newImg = clear;
      },
      updateNewImgData(data) {
        this.newImg.data = data;
      },
      handleUpdateImageSizes(iw, ih) {
        this.originalWidth = iw;
        this.originalHeight = ih;
      },
    },
    watch: {
      state(newValue) {
        if (newValue === "modal") {
          this.closeModals();
          this.isShowModal = true;
        }
        if (newValue === "pipette") {
          this.closeModals();
          this.isShowPanel = true;
        }
        if (newValue === "save") {
          this.handleConfirm();
        }
        if (newValue === "correct") {
          this.closeModals();
          this.isShowCorrection = true;
        }
        if (newValue === "filter") {
          this.closeModals();
          this.isShowFiltration = true;
        }
        if (!newValue) {
          this.closeModals();
        }
      },
      img() {
        this.iw = null;
        this.ih = null;
        this.newiw = null;
        this.newih = null;
        this.scale = 100;
        this.imgLoaded = false; // Сбрасываем флаг загрузки
  
        this.origImg = new Image();
        this.origImg.src = this.img;
        this.origImg.onload = () => {
          this.newImg = this.origImg;
          this.updateImageSizes(this.origImg.width, this.origImg.height);
        };
      },
      isShowModal(newValue) {
        if (newValue) {
          this.updateModal();
        }
      },
    },
  });
  </script>
  
  <style scoped lang="scss">
  .editor {
    flex: 1;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    height: 100vh;
    background-color: #2e2e2e;
    background-size: cover; 
    background-position: center; 
    background-repeat: no-repeat; 
  }
  
  .modal {
    padding: 1px;
    display: flex;
    flex-direction: column;
    gap: 2px;
    background: #202020;
    color:#ffffff;
  }
  .button-image {
    width: 24px;
    height: 24px;
  }
  
  .confirm-button {
    color: #fff;
    cursor: pointer;
    padding: 10px 15px;
    border: none;
    border-radius: 20px;
    background: transparent;
    display: block;
    transition: 0.3s;
    &:hover {
      background: #2e2e2e;
      border-color: #2e2e2e;
    }
  }
  
  .measurements-wrapper {
    display: flex;
    gap: 1px;
  }
  
  .sizes {
    display: flex;
    gap: 1px;
    margin-top: 10px;
  }
  
  .custom-checkbox {
    position: relative;
    display: inline-block;
  
    input,
    .checkbox-image {
      width: 20px;
      height: 20px;
      cursor: pointer;
    }
  
    input[type="checkbox"] {
      position: absolute;
      opacity: 0;
      cursor: pointer;
      &:hover {
        border-color: #ffffff;
      }
    }
  }
  
  .input-with-prefix {
    position: relative;
    input {
      height: 25px;
      width: 75px;
      padding-left: 25px;
      padding-right: 25px;
      border-radius: 10px;
      border: none;
    }
  
    label {
      color: #848484;
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      padding: 0px 5px;
    }
  }
  
  input[type="number"]::-webkit-inner-spin-button,
  input[type="number"]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
  }
  
  .interpolation {
    display: flex;
    gap: 10px;
    margin-top: 10px;
  }
  
  .sizes-counter {
    margin-top: 10px;
    display: flex;
    gap: 10px;
    justify-content: space-between;
  }

  select {
    margin-right: 10px;
    width: 120px;
    padding: 5px 15px;
    margin-left: 5px;
    color: #e2e2e2;
    background-color: #2c2c2c;
    border: 1px solid #e2e2e2;
    border-radius: 10px;
  cursor: pointer;
  }
  </style>