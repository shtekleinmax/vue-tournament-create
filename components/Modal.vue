<script>
export default {
  name: "Modal",
  props: {
    visible: {
      type: Boolean,
      default: false,
    },
    content: {
      type: String,
      default: "",
    },
  },
  data: function () {
    return {
      isModalVisible: this.visible,
      modalContent: this.content,
      modalBig: false,
      action: "",
      location: "",
    };
  },
  methods: {
    showModal: function (ref, big = false, content = "") {
      this.isModalVisible = true;
      this.modalBig = big;

      if (content) {
        this.modalContent = content;
      }

      document.body.classList.add("no-scroll");
    },
    closeModal: function () {
      this.isModalVisible = false;
      document.body.classList.remove("no-scroll");

      if (this.location) {
        window.location = this.location;
      }
    },
  },
};
</script>

<template>
  <transition name="modal">
    <div v-if="isModalVisible" class="overlay" @click.self="closeModal">
      <div class="modal" :class="{ big: modalBig }">
        <div class="modal-close" @click="closeModal">
          <svg-icon icon="close"></svg-icon>
        </div>
        <slot name="title"></slot>
        <slot name="content"
          ><div class="modal-content" v-html="modalContent"></div
        ></slot>
      </div>
    </div>
  </transition>
</template>
