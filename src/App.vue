<template>
  <div>
    <AudioDataTable
      :participants="participants"
      :audio-segments="audioSegments"
      :talk-count="participantsTalkCount"
      :average-times="participantsAverageTime"
      :min-max-contribution="participantsContribution"
      @toggleParticipant="onToggleParticipant($event)"
      @addParticipant="onAddParticipant($event)"
    />
  </div>
</template>

<script>
import vueTime from "vue-time";
import AudioDataTable from "./components/AudioDataTable.vue";

export default {
  name: "App",
  components: {
    AudioDataTable,
    vueTime,
  },
  data() {
    return {
      participants: [],
      audioSegments: [],
      hiddenAudioSegments: [],

      participantsAudioTime: {},
      participantsTalkCount: {},
      participantsAverageTime: {},
      participantsContribution: {
        min: {},
        max: {},
      },
    };
  },
  methods: {
    fetchParticipants() {
      return require("./api/participants.json");
    },
    fetchAudioSegments() {
      return require("./api/audio_segments.json");
    },
    calculateData() {
      this.setParticipantsAudioTime();

      this.calculateParticipantsTalkCount();
      this.calculateAverageTime();
      this.calculateMinMaxContribution();
    },
    resetData() {
      this.participantsAudioTime = {};
      this.participantsTalkCount = {};
      this.participantsAverageTime = {};
      this.participantsContribution = {
        min: {},
        max: {},
      };
    },
    calculateParticipantsTalkCount() {
      this.audioSegments.map((segment) => {
        if (segment.hidden) {
          return;
        }

        if (!this.participantsTalkCount[segment.participant_id]) {
          return (this.participantsTalkCount[segment.participant_id] = 1);
        }

        this.participantsTalkCount[segment.participant_id]++;
      });
    },
    calculateAverageTime() {
      for (const key in this.participantsTalkCount) {
        if (Object.hasOwnProperty.call(this.participantsTalkCount, key)) {
          const avarageTime = this.participantsTalkCount[key];
          this.participantsAverageTime[key] =
            this.participantsAudioTime[key] / avarageTime;
        }
      }
    },
    calculateMinMaxContribution() {
      let min = null;
      let max = 0;
      for (const key in this.participantsAudioTime) {
        if (Object.hasOwnProperty.call(this.participantsAudioTime, key)) {
          const talkTime = this.participantsAudioTime[key];
          if (min === null) {
            min = talkTime;
          }
          if (talkTime > max) {
            max = talkTime;
            this.participantsContribution.max = {
              participant_id: key,
              value: this.participantsAudioTime[key],
            };
          } else if (talkTime < min) {
            min = talkTime;
            this.participantsContribution.min = {
              participant_id: key,
              value: this.participantsAudioTime[key],
            };
          }
        }
      }
    },
    setParticipantsAudioTime() {
      this.audioSegments.map((segment) => {
        if (segment.hidden) {
          return;
        }

        if (!this.participantsAudioTime[segment.participant_id]) {
          this.participantsAudioTime[segment.participant_id] = null;
        }

        this.participantsAudioTime[segment.participant_id] +=
          new Date(segment.audio_end).getTime() -
          new Date(segment.audio_start).getTime();
      });
    },
    onToggleParticipant(id) {
      this.audioSegments = this.audioSegments.map((segment) => {
        if (segment.participant_id == id) {
          segment.hidden = segment.hidden ? !segment.hidden : true;
        }

        return segment;
      });

      this.resetData();
      this.calculateData();
    },
    onAddParticipant(participant) {
      this.audioSegments.push(participant);

      this.resetData();
      this.calculateData();
    },
  },
  created() {
    this.participants = this.fetchParticipants();
    this.audioSegments = this.fetchAudioSegments();

    this.calculateData();
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
