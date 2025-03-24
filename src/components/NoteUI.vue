<template>
  <div
    class="note"
    :style="{ left: note.x + 'px', top: note.y + 'px', backgroundColor: note.color }">
    <button class="delete-btn" @click="deleteNote">
      <svg width="14" height="14" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M6 6L18 18M18 6L6 18" stroke="white" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
      </svg>
    </button>
    <div class="mouse" @mousedown="startDrag"></div>
    <textarea
      v-model="text"
      placeholder="Введите текст"
      @input="updateText"
      ref="textarea"
    ></textarea>
  </div>
</template>

<script>
export default {
  name: 'NoteUI',
  props: {
    note: Object,
    index: Number
  },
  data () {
    return {
      text: this.note.text,
      isDragging: false,
      offset: { x: 0, y: 0 },
      selectedNoteIndex: {
        type: Number,
        default: null
      },
      notes: [
        { text: 'Заметка 1', x: 400, y: 400, color: 'red' },
        { text: 'Заметка 2', x: 200, y: 200, color: 'blue' },
        { text: 'Заметка 3', x: 100, y: 100, color: 'yellow' },
        { text: 'Заметка 4', x: 300, y: 300, color: 'green' }
      ]
    }
  },
  watch: {
    note: {
      handler (newVal) {
        this.text = newVal.text
        this.$nextTick(() => this.adjustHeight())
      },
      deep: true
    }
  },
  mounted () {
    this.adjustHeight()
  },
  methods: {
    deleteNote () {
      this.$emit('deleteNote', this.index)
    },
    startDrag (event) {
      this.isDragging = true
      this.offset.x = event.clientX - this.note.x
      this.offset.y = event.clientY - this.note.y
      window.addEventListener('mousemove', this.drag)
      window.addEventListener('mouseup', this.stopDrag)
    },
    drag (event) {
      if (this.isDragging) {
        const newX = event.clientX - this.offset.x
        const newY = event.clientY - this.offset.y
        this.$emit('updatePosition', { x: newX, y: newY })
      }
    },
    stopDrag () {
      this.isDragging = false
      window.removeEventListener('mousemove', this.drag)
      window.removeEventListener('mouseup', this.stopDrag)
    },
    updateText () {
      this.adjustHeight()
      this.$emit('updateText', this.text)
    },
    adjustHeight () {
      this.$nextTick(() => {
        const textarea = this.$refs.textarea
        if (textarea) {
          textarea.style.height = textarea.scrollHeight + 'px'
        }
      })
    }
  }
}
</script>

<style lang="scss">
.note {
  position: absolute;
  width: 200px;
  background-color: #fff8dc;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  padding: 0 16px 16px 16px;

  .delete-btn {
    position: absolute;
    top: -10px;
    right: -10px;
    background: red;
    color: white;
    border: none;
    width: 24px;
    height: 24px;
    border-radius: 50%;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: background 0.3s ease;

    &:hover {
      background: darkred;
    }
  }

  .mouse {
    position: relative;
    top: -10px;
    left: 50%;
    transform: translateX(-50%);
    background-color: rgb(148, 148, 148);
    width: 40px;
    height: 20px;
    border-radius: 3px;
    cursor: pointer;
  }

  textarea {
    width: 100%;
    min-height: 50px;
    border: none;
    outline: none;
    resize: none;
    background: transparent;
    font-size: 14px;
    font-family: Arial, sans-serif;
    overflow: hidden;
  }
}
</style>
