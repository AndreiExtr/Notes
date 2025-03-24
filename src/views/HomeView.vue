<template>
  <div class="home">
    <div class="home__content">
      <canvas ref="canvas" class="canvas" width="1920" height="1080"></canvas>
      <div class="rectangle">
        <div class="square red" @click="createNote('#FEB1B1')"></div>
        <div class="square blue" @click="createNote('#B1D0FE')"></div>
        <div class="square yellow" @click="createNote('#FCFEB1')"></div>
        <div class="square green" @click="createNote('#c2ffc2')"></div>
      </div>

      <NoteUI
        v-for="(note, index) in notes"
        :key="index"
        :note="note"
        :index="index"
        :notes="notes"
        :selectedNoteIndex="selectedNoteIndex"
        @updatePosition="updatePosition(index, $event)"
        @updateText="updateText(index, $event)"
        @deleteNote="deleteNote(index)"
        @updateSelectedNote="updateSelectedNote"
      />
    </div>
  </div>
</template>

<script>
import { toRaw } from 'vue'
import NoteUI from '@/components/NoteUI.vue'

export default {
  name: 'HomeView',
  components: {
    NoteUI
  },
  data () {
    return {
      notes: [
      ],
      connections: [],
      selectedNoteIndex: null
    }
  },
  created () {
    this.loadNotes()
  },
  methods: {
    loadNotes () {
      const savedNotes = localStorage.getItem('notes')
      if (savedNotes) {
        try {
          const parsedNotes = JSON.parse(savedNotes)
          this.notes = parsedNotes.map(note => ({
            ...note,
            connections: note.connections || []
          }))
        } catch (error) {
          console.error('Ошибка при загрузке заметок:', error)
        }
      }
    },
    saveNotes () {
      localStorage.setItem('notes', JSON.stringify(this.notes))
    },
    updateSelectedNote (index) {
      this.selectedNoteIndex = index
    },
    createNote (color) {
      const x = Math.random() * 300
      const y = Math.random() * 300
      const newNote = { x, y, color, text: '', connections: [] }
      this.notes.push(newNote)
      this.saveNotes()
    },
    updatePosition (index, newPosition) {
      const contentRect = this.$el.querySelector('.home__content').getBoundingClientRect()
      const x = Math.max(0, Math.min(newPosition.x, contentRect.width - 200))
      const y = Math.max(0, Math.min(newPosition.y, contentRect.height - 110))

      this.notes[index] = { ...this.notes[index], x, y }
      this.saveNotes()
    },
    updateText (index, newText) {
      this.notes[index].text = newText
      this.saveNotes()
    },
    deleteNote (index) {
      this.notes.splice(index, 1)
      this.saveNotes()
    },
    drawConnections () {
      const canvas = this.$refs.canvas
      const ctx = canvas.getContext('2d')

      ctx.clearRect(0, 0, canvas.width, canvas.height)

      this.notes.forEach((note, index) => {
        if (note.connections && Array.isArray(note.connections) && note.connections.length > 0) {
          note.connections.forEach(connectedIndex => {
            const fromNote = toRaw(this.notes[index])
            const toNote = toRaw(this.notes[connectedIndex])

            if (!fromNote || !toNote) {
              console.error('Одна из заметок не найдена:', fromNote, toNote)
              return
            }

            const fromX = fromNote.x + 100
            const fromY = fromNote.y + 50
            const toX = toNote.x + 100
            const toY = toNote.y + 50

            ctx.beginPath()
            ctx.moveTo(fromX, fromY)
            ctx.lineTo(toX, toY)
            ctx.strokeStyle = fromNote.color || '#ff5a00'
            ctx.lineWidth = 2
            ctx.stroke()
          })
        } else {
          console.log(`Заметка ${index} не имеет связей`)
        }
      })
    }
  }
}
</script>

<style lang="scss">
.home {
  height: 100vh;
  padding: 24px;
  background-color: #ffffff;

  &__content {
    position: relative;
    height: 100%;
    background-color: #F6F6F6;
    border-radius: 16px;
    box-shadow: 0px 4px 20px 10px rgba(0, 0, 0, 0.09);
    overflow: hidden;

    .canvas {
      border: 1px solid black;
      width: 100%;
      height: 100%;
    }

    // Сетка из точек
    &::before {
      content: "";
      display: grid;
      grid-template-columns: repeat(50, 1fr);
      grid-template-rows: repeat(60, 1fr);
      gap: 16px;
      pointer-events: none;
      z-index: 0;

      background: radial-gradient(circle, #D9D9D9 2px, transparent 2px);
      background-size: 16px 16px; // Шаг сетки точек
      background-position: center;
    }

    .rectangle {
      position: absolute;
      display: flex;
      flex-direction: row;
      bottom: 16px;
      left: 16px;
      width: 160px;
      height: 60px;
      background-color: #ffffff;
      border-radius: 8px;
      border: 1px solid #D9D9D9;
      display: flex;
      justify-content: space-around;
      align-items: center;
      padding: 8px;

      .square {
        width: 24px;
        height: 24px;
        border-radius: 4px;
        cursor: pointer;

        &.red {
          background-color: #FEB1B1;
        }

        &.blue {
          background-color: #B1D0FE;
        }

        &.yellow {
          background-color: #FCFEB1;
        }

        &.green {
          background-color: #c2ffc2;
        }
      }
    }
  }
}
</style>
