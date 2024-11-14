<template>
  <div class="filtration-modal">
    <div class="panel-header">
      <button class="close-button" @click="close">
        <img src="../assets/cross.png" alt="Close" />
      </button>
    </div>
    <div class="filtrations-graph">
      <select class="filtration-preset" @change="applyPreset($event.target.value)">
        <option value="identity">Тождественное отображение</option>
        <option value="sharpen">Повышение резкости</option>
        <option value="gaussian">Фильтр Гаусса</option>
        <option value="boxBlur">Прямоугольное размытие</option>
      </select>
      <div class="filtration-inputs">
        <div class="point-inputs">
          <div class="point-inputs-row" v-for="rowIndex in [0, 1, 2]" :key="rowIndex">
            <div v-for="colIndex in [0, 1, 2]" :key="colIndex" class="point-with-prefix">
              <input :value="kernel[rowIndex][colIndex]" @change="(e) => updateKernel(e, rowIndex, colIndex)"
                :disabled="previewEnabled" />
            </div>
          </div>
        </div>
      </div>
      <div class="filtration-actions">
        <div class="custom-checkbox">
          <input type="checkbox" v-model="previewEnabled" />
          <img v-if="previewEnabled" class="checkbox-image" src="../assets/eye.png" alt="Lock" />
          <img v-else class="checkbox-image" src="../assets/closed-eye.png" alt="Unlock" />
        </div>
        <button class="filtration-button" @click="applyFiltration">
          Применить
        </button>
        <button class="filtration-button" @click="resetFiltration">
          Сбросить
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "FiltrationModal",
  props: {
    state: String,
    canvasRef: Object,
    origImg: Object,
    newImg: Object,
    dx: Number,
    dy: Number,
    ih: Number,
    iw: Number,
  },
  data() {
    return {
      previewEnabled: false,
      kernel: [
        [0, 0, 0],
        [0, 1, 0],
        [0, 0, 0],
      ],
    };
  },
  methods: {
    close() {
      this.$emit("close");
    },
    applyPreset(preset) {
      switch (preset) {
        case "identity":
          this.kernel = [
            [0, 0, 0],
            [0, 1, 0],
            [0, 0, 0],
          ];
          break;
        case "sharpen":
          this.kernel = [
            [0, -1, 0],
            [-1, 5, -1],
            [0, -1, 0],
          ];
          break;
        case "gaussian":
          this.kernel = [
            [1, 2, 1],
            [2, 4, 2],
            [1, 2, 1],
          ];
          break;
        case "boxBlur":
          this.kernel = [
            [1, 1, 1],
            [1, 1, 1],
            [1, 1, 1],
          ];
          break;
        default:
          break;
      }
    },
    updateKernel(event, rowIndex, colIndex) {
      const num = +event.target.value;

      if (!isNaN(num)) {
        this.kernel[rowIndex][colIndex] = num;
      } else {
        this.kernel[rowIndex][colIndex] += 1;
        this.kernel[rowIndex][colIndex] -= 1;
      }
    },
    calculateFiltration() {
      const ctx = this.canvasRef?.getContext("2d");
      const imageData = ctx.getImageData(this.dx, this.dy, this.iw, this.ih);
      const newData = new Uint8ClampedArray(imageData.data.length);

      // Обработка краев (padding)
      const paddedData = this.padImageData(
        imageData.data,
        imageData.width,
        imageData.height
      );

      // Рассчитать сумму ядра
      const kernelSum =
        this.kernel.flat().reduce((sum, val) => sum + val, 0) || 1;

      // Свертка по каналам
      for (let y = 0; y < imageData.height; y++) {
        for (let x = 0; x < imageData.width; x++) {
          for (let c = 0; c < 3; c++) {
            // Каналы: R, G, B (без A)
            const outputIndex = (y * imageData.width + x) * 4 + c;
            let sum = 0;
            for (let ky = 0; ky < 3; ky++) {
              for (let kx = 0; kx < 3; kx++) {
                const inputIndex =
                  ((y + ky) * (imageData.width + 2) + (x + kx)) * 4 + c;
                sum += paddedData[inputIndex] * this.kernel[ky][kx];
              }
            }
            // Нормализация и ограничение значений пикселей в диапазоне 0-255
            newData[outputIndex] = Math.min(Math.max(sum / kernelSum, 0), 255);
          }
          // Сохранение значения альфа-канала
          newData[(y * imageData.width + x) * 4 + 3] =
            imageData.data[(y * imageData.width + x) * 4 + 3];
        }
      }

      imageData.data.set(newData);
      ctx.putImageData(imageData, this.dx, this.dy);
    },

    padImageData(data, width, height) {
      const paddedWidth = width + 2;
      const paddedHeight = height + 2;
      const paddedData = new Uint8ClampedArray(paddedWidth * paddedHeight * 4);

      // Копирование исходных данных с добавлением отступов
      for (let y = 0; y < height; y++) {
        for (let x = 0; x < width; x++) {
          const inputIndex = (y * width + x) * 4;
          const outputIndex = ((y + 1) * paddedWidth + (x + 1)) * 4;
          paddedData.set(
            data.subarray(inputIndex, inputIndex + 4),
            outputIndex
          );
        }
      }

      // Копирование краёв
      for (let y = 0; y < paddedHeight; y++) {
        for (let x = 0; x < paddedWidth; x++) {
          const outputIndex = (y * paddedWidth + x) * 4;
          if (
            x === 0 ||
            x === paddedWidth - 1 ||
            y === 0 ||
            y === paddedHeight - 1
          ) {
            const nearestX = Math.min(Math.max(x, 1), paddedWidth - 2);
            const nearestY = Math.min(Math.max(y, 1), paddedHeight - 2);
            const nearestIndex = (nearestY * paddedWidth + nearestX) * 4;
            paddedData.set(
              paddedData.subarray(nearestIndex, nearestIndex + 4),
              outputIndex
            );
          }
        }
      }

      return paddedData;
    },
    applyFiltration() {
      this.$emit("close");
      this.calculateFiltration();
    },
    resetFiltration() {
      this.$emit("revertNewImg");
      this.previewEnabled = false;
      this.applyPreset("identity");
    },
  },
  watch: {
    previewEnabled(newVal) {
      if (newVal) {
        this.calculateFiltration();
      } else {
        this.resetFiltration();
      }
    },
  },
});
</script>

<style scoped lang="scss">
.filtration-modal {
  position: absolute;
  top: 40%;
  right: 20px;
  width: 250px;
  height: 180px;
  background-color: #202020;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  padding: 20px;

  .panel-header {
    display: flex;
    justify-content: flex-end;

    .close-button {
      cursor: pointer;
      background: transparent;
      border: none;
      img {
        height: 15px;
      }
    }
  }

  .filtrations-graph {
    font-size: small;
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    
  }

  .filtration-preset {
    border: 1px solid black;
  }

  .filtration-inputs {
    display: flex;
    flex-direction: column;
    align-items: center;

    .point-inputs {
      display: flex;
      flex-direction: column;
      gap: 3px;

      .point-inputs-row {
        display: flex;
        gap: 3px;
      }

      .point-with-prefix {
        position: relative;

        input {
          height: 25px;
        width: 40px;
        border-radius: 10px;
        border: none;
        padding-left: 40px;
        }
      }
    }
  }

  .filtration-actions {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    gap: 3px;

    .filtration-button {
      color: #fff;
      cursor: pointer;
      padding: 10px 15px;
      border: none;
      border-radius: 20px;
      background: transparent;
      width: auto;
      background: #202020;
      transition: 0.2s;

      &:hover {
        background: #646464;
      }
    }
  }
}

.custom-checkbox {
  position: relative;
  display: inline-block;
  height: 20px;

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
      border-color: #81cffb;
    }
  }
}
select {
    margin-right: 10px;
    width: 220px;
    padding: 5px 15px;
    margin-left: 5px;
    color: #e2e2e2;
    background-color: #2c2c2c;
    border-radius: 10px;
    border-color: #fff;
  cursor: pointer;
  }
</style>