<template>
    <transition name="modal-fade">
      <div class="modal-window">
        <div
          class="modal"
          role="dialog"
          aria-labelledby="modalTitle"
          aria-describedby="modalDescription"
        >
          <header class="modal-header" id="modalTitle">
            <slot name="header"></slot>
            <button
              type="button"
              class="btn-close"
              @click="close"
              aria-label="Close modal"
            >
            <img src="../assets/cross.png" alt="Close" />
            </button>
          </header>
  
          <section class="modal-body" id="modalDescription">
            <slot name="body"></slot>
          </section>
  
          <footer class="modal-footer">
            <slot name="footer"></slot>
          </footer>
        </div>
      </div>
    </transition>
  </template>
  
  <script>
  export default {
    name: "ModalWindow",
    methods: {
      close() {
        this.$emit("close");
      },
    },
  };
  </script>
  
  <style>
  .modal-window {
    width: 350px;
    height: 320px;
    background-color: #202020;
    position: absolute;
    top: 35%;
    right: 20px;
    display: flex;
    justify-content: center;
    gap: 30px;
    align-items: center;
    border-radius: 20px;
    
    
  }
  
  .modal {
    overflow-x: auto;
    display: flex;
    flex-direction: column;
  }
  
  .modal-header,
  .modal-footer {
    background: #202020;
    padding: 20px;
    display: flex;
  }
  
  .modal-header {
    position: relative;
    border-bottom: 1px solid #eeeeee;
    color: #ececec;
    justify-content: space-between;
  }
  
  .modal-footer {
    border-top: 1px solid #eeeeee;
    flex-direction: column;
  }
  
  .modal-body {
    padding: 10px;
    position: relative;
    display: flex;
    flex-direction: column;
  
  }
  
  .btn-close {
    position: absolute;
    top: 0;
    right: 0;
    border: none;
    font-size: 20px;
    padding: 10px 15px;
    cursor: pointer;
    font-weight: bold;
    color: #4f4f4f;
    background: transparent;
  }

  .btn-close img{
    position: absolute;
    width: 15px;
    height: 15px;
  }
  
  .modal-fade-enter,
  .modal-fade-leave-to {
    opacity: 0;
  }
  
  .modal-fade-enter-active,
  .modal-fade-leave-active {
    transition: opacity 0.5s ease;
  }
  </style>