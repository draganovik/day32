<template>
  <Modal ref="modal">
    <div class="form">
      <input
        type="text"
        placeholder="Title"
        v-model="event.summary"
        ref="finput"
        tabindex="1"
      >
      <input
        type="button"
        class="btn"
        style="width: 70%; box-sizing: border-box"
        ref="adbtn"
        @click="allDay()"
        value="Set without time"
        tabindex="-1"
      >
      <input
        type="datetime-local"
        ref="sdate"
        v-model="event.start.dateTime"
        tabindex="2"
      >
      <input
        type="datetime-local"
        ref="edate"
        class="tdate"
        v-model="event.end.dateTime"
        tabindex="3"
      >
      <input
        type="date"
        ref="adate"
        class="noned"
        v-model="event.start.date"
        tabindex="4"
      >
      <input
        type="text"
        placeholder="Location"
        v-model="event.location"
        tabindex="5"
      >
      <select
        name="color"
        id="clr"
        v-model="event.colorId"
        tabindex="6"
      >
        <option value="undefined">
          Default
        </option>
        <option value="1">
          Levander
        </option>
        <option value="2">
          Sage
        </option>
        <option value="3">
          Grape
        </option>
        <option value="4">
          Flamingo
        </option>
        <option value="5">
          Banana
        </option>
        <option value="6">
          Tangerine
        </option>
        <option value="7">
          Peacock
        </option>
        <option value="8">
          Graphite
        </option>
        <option value="9">
          Blueberry
        </option>
        <option value="10">
          Basil
        </option>
        <option value="11">
          Tomato
        </option>
      </select>
      <textarea
        name="desc"
        rows="8"
        cols="4"
        placeholder="Description"
        v-model="event.description"
        tabindex="7"
      />
      <div
        style="
          display: flex;
          justify-content: space-between;
          width: 70%;
          margin: auto;
        "
      >
        <button
          class="btn ac"
          @click="toggleModal()"
        >
          Cancel
        </button>
        <button
          class="btn ac"
          @click="updateEvent()"
          tabindex="8"
          style="background-color: var(--accent); color: white"
        >
          Save
        </button>
      </div>
      <br>
    </div>
  </Modal>
</template>

<script>
import Modal from '@/components/TemplateModal.vue'
export default {
  components: {
    Modal
  },
  methods: {
    toggleModal: function () {
      this.$refs.modal.toggleModal()
    },
    validateTime: function () {
      if (new Date(this.event.start.date)) {
        if (new Date(this.event.start.date) <= new Date(this.event.end.date)) {
          return true
        }
      }
      if (new Date(this.event.start.dateTime)) {
        if (
          new Date(this.event.start.dateTime) <=
          new Date(this.event.end.dateTime)
        ) {
          return true
        }
      }
      return false
    },
    updateEvent: function () {
      if (this.event.summary !== '' && this.validateTime()) {
        this.$gapi
          ._libraryInit('client', {
            discoveryDocs: [
              'https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest'
            ]
          })
          .then((client) => {
            var ev = {
              calendarId: 'primary',
              eventId: this.event.id,
              resource: this.event
            }
            if (this.$refs.adate.classList.contains('noned')) {
              delete this.event.start.date
              delete this.event.end.date
              this.event.start.dateTime = new Date(
                this.event.start.dateTime
              ).toISOString()
              this.event.end.dateTime = new Date(
                this.event.end.dateTime
              ).toISOString()
              return client.calendar.events.update(ev)
            }
            delete this.event.start.dateTime
            delete this.event.end.dateTime
            this.event.start.date = this.event.start.date.substring(0, 10)
            this.event.end.date = this.event.start.date
            return client.calendar.events.update(ev)
          })
          .then(this.toggleModal())
          .then(
            setTimeout(function () {
              // TODO Replace logic with vuex no-reload solution
              location.reload()
            }, 1500)
          )
      }
    },
    allDay: function () {
      if (this.$refs.adate.classList.contains('noned')) {
        /* TODO Date to time convert
        if (this.event.start.dateTime) {
          this.event.start.date = this.event.start.dateTime.substring(0, 10)
          this.event.end.date = this.event.end.dateTime.substring(0, 10)
        } */
        this.$refs.adate.classList.remove('noned')
        this.$refs.adbtn.classList.add('actbt')
        this.$refs.sdate.classList.add('noned')
        this.$refs.edate.classList.add('noned')
      } else {
        /* TODO Date to time convert
        if (this.event.start.dateTime === undefined) {
          this.event.start.dateTime = new Date(new Date(this.event.start.date).getTime() - (new Date(this.event.start.date).getTimezoneOffset() * 60000)).toISOString()
          this.event.end.dateTime = new Date(new Date(this.event.end.date).getTime() - (new Date(this.event.end.date).getTimezoneOffset() * 60000)).toISOString()
        } */
        this.$refs.adate.classList.add('noned')
        this.$refs.adbtn.classList.remove('actbt')
        this.$refs.sdate.classList.remove('noned')
        this.$refs.edate.classList.remove('noned')
      }
    }
  },
  props: {
    toggle: Number,
    event: {
      type: Object,
      default: function () {
        return {
          summary: '',
          location: undefined,
          description: undefined,
          colorId: 0,
          start: {
            dateTime: new Date(Date.now()).toISOString()
          },
          end: {
            dateTime: new Date(Date.now()).toISOString()
          }
        }
      }
    }
  },
  watch: {
    toggle: function (update, outdate) {
      if (update !== undefined) {
        if (this.event.summary !== '') {
          if (
            (this.event.end.dateTime === undefined &&
              !this.$refs.edate.classList.contains('noned')) ||
            (this.event.end.dateTime !== undefined &&
              this.$refs.edate.classList.contains('noned'))
          ) {
            this.allDay()
          }
          if (!this.$refs.edate.classList.contains('noned')) {
            this.event.start.dateTime = new Date(
              new Date(this.event.start.dateTime).getTime() -
                new Date(this.event.start.dateTime).getTimezoneOffset() * 60000
            )
              .toISOString()
              .substring(0, 16)
            this.event.end.dateTime = new Date(
              new Date(this.event.end.dateTime).getTime() -
                new Date(this.event.end.dateTime).getTimezoneOffset() * 60000
            )
              .toISOString()
              .substring(0, 16)
          }
          this.toggleModal()
        }
      }
    }
  }
}
</script>
