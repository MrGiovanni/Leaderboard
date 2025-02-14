<template>
  <div class="custom-select" ref="dropdown" @click="toggleDropdown">
    <div class="selected-option">{{ selectedOptionLabel }}</div>
    <transition name="dropdown">
      <ul v-show="open" class="options">
        <li
          v-for="option in options"
          :key="option.value"
          @click.stop="selectOption(option)"
        >
          {{ option.label }}
        </li>
      </ul>
    </transition>
  </div>
</template>

<script>
export default {
  name: "AnimatedDropdown",
  props: {
    options: {
      type: Array,
      default: () => [],
    },
    modelValue: {
      type: [String, Number],
      default: null,
    },
  },
  data() {
    return {
      open: false,
    };
  },
  computed: {
    selectedOptionLabel() {
      const selected = this.options.find(
        (opt) => opt.value === this.modelValue
      );
      return selected ? selected.label : "Select...";
    },
  },
  methods: {
    toggleDropdown() {
      this.open = !this.open;
    },
    selectOption(option) {
      this.$emit("update:modelValue", option.value);
      this.open = false;
    },
    handleClickOutside(event) {
      if (this.$refs.dropdown && !this.$refs.dropdown.contains(event.target)) {
        this.open = false;
      }
    },
  },
  mounted() {
    document.addEventListener("click", this.handleClickOutside);
  },
  beforeUnmount() {
    document.removeEventListener("click", this.handleClickOutside);
  },
};
</script>

<style scoped>
.custom-select {
  position: relative;
  width: 250px;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  background-color: #ffffff;
  cursor: pointer;
  user-select: none;
}

.selected-option {
  padding: 8px 12px;
}

.options {
  list-style: none;
  padding: 0;
  margin: 0;
  position: absolute;
  width: 100%;
  top: 100%;
  left: 0;
  border: 1px solid #e0e0e0;
  border-top: none;
  border-radius: 0 0 4px 4px;
  background-color: #ffffff;
  overflow: hidden;
  z-index: 10;
}

.options li {
  padding: 8px 12px;
  transition: background-color 0.2s ease;
}

/* 마우스 오버 시 배경색과 bold 처리 */
.options li:hover {
  background-color: #f8f8f8;
  font-weight: bold;
}

/* Vue transition 클래스 (드롭다운 애니메이션 효과) */
.dropdown-enter-active,
.dropdown-leave-active {
  transition: all 0.3s ease;
}
.dropdown-enter-from,
.dropdown-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
.dropdown-enter-to,
.dropdown-leave-from {
  opacity: 1;
  transform: translateY(0);
}
</style>
