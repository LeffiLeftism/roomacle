<template>
  <div @click="openModal()" v-show="this.dateChecked" id="post">
    <div
      class="lfpost"
      :class="[
        this.postContent.pinned || this.postContent.timerActive === true
          ? 'pinned'
          : 'notpinned',
      ]"
    >
      <span class="textline">
        <!--span v-if="this.postContent.time">| {{ this.postContent.time }}</span-->
        <span v-if="this.postContent.title">{{ this.postContent.title }}</span>
        <span
          class="timer"
          :id="this.index + 'timer'"
          v-if="this.postContent.timerActive"
          >{{ this.postContent.timerCountdown }}</span
        >
      </span>
    </div>
  </div>
</template>

<script>
import AnnouncementsPopupVue from "./AnnouncementsPopup.vue";

export default {
  data() {
    return {
      timer_started: false,
      timer_end: false,
      dateChecked: false,
    };
  },
  props: {
    postContent: Object,
    index: Number,
  },
  computed: {
    tomorrow: function () {
      //Gibt den künftigen Tag zurück
      let date = new Date(
        this.$store.state.calendar.today.year,
        this.$store.state.calendar.today.month,
        this.$store.state.calendar.today.day,
        0,
        0,
        0,
        0
      );
      date.setDate(date.getDate() + 1);
      return date;
    },
    postDate: function () {
      //Gibt das Startdatum der Benachrichtung zurück, da Datum und Uhrzeit in getrenten Variablen gespeichert sind
      let postDate = new Date(this.postContent.date);
      let postHour = this.postContent.time;
      postHour = postHour.slice(0, 2);
      let postMinute = this.postContent.time;
      postMinute = postMinute.slice(3, 5);
      postHour = Number(postHour);
      postMinute = Number(postMinute);
      postDate.setHours(postHour, postMinute, 0, 0);
      return postDate;
    },
    postEndDate: function () {
      //Gibt das Enddatum der Benachrichtitung zurück
      let postEndDate = new Date(this.postContent.countDownDate);
      return postEndDate;
    },
  },
  methods: {
    checkDate() {
      //Überprüft, ob die Benachrichtigung angezeigt werden soll, anhand von Datum und Uhrzeit
      let now = new Date();
      if (
        !this.postContent.pinned ||
        (this.postContent.pinned && !this.postContent.timerActive) ||
        (this.postContent.pinned &&
          this.postContent.timerActive &&
          this.postDate < now &&
          this.postEndDate > now &&
          !this.postContent.timer_ended)
      ) {
        //console.log("Wird angezeigt:");
        //console.log(this.postContent.title);
        this.dateChecked = true;
        return true;
      } else {
        //console.log(this.postContent.title);
        //console.log("stop");
        this.dateChecked = false;
        return false;
      }
    },
    openModal() {
      //Öffnet das Popup der Benachrichtigungen
      try {
        this.$modal.show(
          AnnouncementsPopupVue,
          {
            postContent: this.postContent,
          },
          { height: "auto" }
        );
      } catch (err) {
        console.log("Error on AnnouncementsPopup.");
      }
    },
    setTimer() {
      //Setzt die Restzeit des Timers und schreibt diese neben die Benachrichtigung

      //Bestimme Abstand von Endzeit und aktueller Zeit
      var distance = this.postEndDate.getTime() - new Date();

      //Bestimme Tage, Stunden, Minuten und Sekunden des Abstands
      var days = Math.floor(distance / (1000 * 60 * 60 * 24));
      var hours = Math.floor(
        (distance % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)
      );
      var minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
      var seconds = Math.floor((distance % (1000 * 60)) / 1000);

      hours = this.$parent.$parent.fillUpTens(hours);
      minutes = this.$parent.$parent.fillUpTens(minutes);
      seconds = this.$parent.$parent.fillUpTens(seconds);

      //Zeige den Abstand neben der Benachrichtigung an
      let text = "";
      if (days > 0) {
        text += days + "d ";
      }
      text += hours + "h " + minutes + "m " + seconds + "s ";

      this.postContent.timerCountdown = text;

      try {
        document.getElementById(this.index + "timer").innerHTML = text;
      } catch (error) {
        //console.log(error);
      }

      //Wenn der Timer ausgelaufen wird dies hinterlegt
      if (distance < 0) {
        this.postContent.timer_ended = true;
        this.checkDate();
        //this.postContent.timerActive = false;
      }
    },
  },
  mounted() {
    this.checkDate();
    //Überprüft ob Benachrichtigung einen Timer hat
    if (
      this.postContent.pinned &&
      this.postEndDate > new Date() &&
      this.dateChecked &&
      !this.postContent.timer_started
    ) {
      //Hinterlegt die Anzahl der laufenden Timer
      this.$store.state.timer_running++;
      this.postContent.timer_started = true;
      this.postContent.timer_ended = false;

      //Startet die Aktualisierung jede 1000ms des Timers
      let x = setInterval(() => {
        if (this.postContent.timer_started && !this.postContent.timer_ended) {
          this.setTimer();
        } else if (this.postContent.timer_ended) {
          //Wenn Timer endet, wird die regelmäßige Aktualisierung beendet
          clearInterval(x);
          this.$store.state.timer_running--;
        }
      }, 1000);
    }
  },
};
</script>

<style>
.lfpost {
  max-width: 100%;
  padding: 0 7px;
  margin-bottom: 5px;
  text-align: left;
}
.textline {
  font-size: 1em;
  font-weight: bold;
}
.timer {
  float: right;
}
</style>