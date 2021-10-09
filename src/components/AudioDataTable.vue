<template>
  <div class="audio-data d-flex-center">
    <div class="audio-data__table d-flex-column">
      <h3>Audio data</h3>
      <table cellpadding="10" border="1">
        <tr>
          <th>User</th>
          <th>Audio start (hh.mm.ss)</th>
          <th>Audio end (hh.mm.ss)</th>
        </tr>
        <tr v-for="(item, index) in filteredAudioSegments" :key="index">
          <td>{{ participantName(item.participant_id) }}</td>
          <td>{{ audioTime(item.audio_start) }}</td>
          <td>{{ audioTime(item.audio_end) }}</td>
        </tr>
      </table>

      <h3>Add participant</h3>
      <table cellpadding="10" border="1">
        <tr>
          <th>User</th>
          <th>Audio start (hh.mm.ss)</th>
          <th>Audio end (hh.mm.ss)</th>
          <th>Action</th>
        </tr>
        <tr>
          <td>
            <select v-model="selectedParticipantId">
              <option
                v-for="(item, index) in participants"
                :key="index"
                :value="item.participant_id"
              >
                {{ item.name }}
              </option>
            </select>
          </td>
          <td>
            <input type="time" step="2" v-model="selectedAudioStart" />
          </td>
          <td>
            <input type="time" step="2" v-model="selectedAudioEnd" />
          </td>
          <td><button @click="onAddParticipant">Add</button></td>
        </tr>
      </table>
    </div>

    <div class="audio-data__speak-amount d-flex-column-between">
      <div class="d-flex-column">
        <h3>Тhe Participants talk count during a call</h3>
        <table cellpadding="10" border="1">
          <tr>
            <th>User</th>
            <th>Talk count</th>
          </tr>
          <tr v-for="(value, key) in talkCount" :key="key">
            <td>{{ participantName(key) }}</td>
            <td>{{ value }}</td>
          </tr>
        </table>
      </div>
      <div class="d-flex-column">
        <h3>Тhe Participants avarage time during a call</h3>
        <table cellpadding="10" border="1">
          <tr>
            <th>User</th>
            <th>Average time</th>
          </tr>
          <tr v-for="(value, key) in averageTimes" :key="key">
            <td>{{ participantName(key) }}</td>
            <td>{{ audioTime(value) }}</td>
          </tr>
        </table>
      </div>
      <div class="d-flex-column">
        <h3>Тhe Participants contribution</h3>
        <table cellpadding="10" border="1">
          <tr>
            <th>Min</th>
            <th>Max</th>
          </tr>
          <tr>
            <td>
              {{ participantName(minMaxContribution.min.participant_id) }} ({{
                audioTime(minMaxContribution.min.value)
              }})
            </td>
            <td>
              {{ participantName(minMaxContribution.max.participant_id) }} ({{
                audioTime(minMaxContribution.max.value)
              }})
            </td>
          </tr>
        </table>
      </div>
      <div class="d-flex-column">
        <h3>Show/Hide Participants</h3>
        <table cellpadding="10" border="1">
          <tr>
            <th>User</th>
            <th>Is shown</th>
          </tr>
          <tr v-for="(participant, index) in participants" :key="index">
            <td>{{ participant.name }}</td>
            <td>
              <div class="d-flex-center-center">
                <input
                  type="checkbox"
                  checked
                  @change="
                    $emit('toggleParticipant', participant.participant_id)
                  "
                />
              </div>
            </td>
          </tr>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "AudioDataTable",
  data() {
    return {
      filteredAudioSegments: [],
      selectedParticipantId: null,
      selectedAudioStart: null,
      selectedAudioEnd: null,
    };
  },
  props: {
    participants: {
      type: [Object, Array],
      required: true,
      default: [],
    },
    audioSegments: {
      type: [Object, Array],
      required: true,
      default: [],
    },
    talkCount: {
      type: Object,
      required: true,
    },
    averageTimes: {
      type: Object,
      required: true,
    },
    minMaxContribution: {
      type: Object,
      required: true,
    },
  },
  methods: {
    participantName(id) {
      return this.participants.find((p) => p.participant_id == id)?.name;
    },
    audioTime(miliseconds) {
      if (!miliseconds) {
        return;
      }

      const date = new Date(miliseconds);
      let hours = date.getUTCHours();
      let minutes = date.getUTCMinutes();
      let seconds = date.getUTCSeconds();
      hours = hours < 10 ? "0" + hours : hours;
      minutes = minutes < 10 ? "0" + minutes : minutes;
      seconds = seconds < 10 ? "0" + seconds : seconds;
      return `${hours}:${minutes}:${seconds}`;
    },
    onAddParticipant() {
      if (
        !this.selectedAudioEnd ||
        !this.selectedAudioStart ||
        !this.selectedParticipantId
      ) {
        return alert("All fields are required");
      }

      const date = new Date();
      date.setUTCHours(...this.selectedAudioEnd.split(":"));
      const audio_end = date.toUTCString();
      date.setUTCHours(...this.selectedAudioStart.split(":"));
      const audio_start = date.toUTCString();

      this.$emit("addParticipant", {
        audio_end,
        audio_start,
        participant_id: this.selectedParticipantId,
      });
    },
  },
  watch: {
    audioSegments: {
      immediate: true,
      handler(segments) {
        this.filteredAudioSegments = segments.filter((as) => !as.hidden);
      },
    },
  },
};
</script>

<style lang="scss" scoped>
.d-flex-center {
  display: flex;
  justify-content: center;
}
.d-flex-center-center {
  display: flex;
  justify-content: center;
  align-items: center;
}
.d-flex-between {
  display: flex;
  justify-content: space-between;
}
.d-flex-column {
  display: flex;
  flex-direction: column;
}
.d-flex-column-between {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.audio-data {
  width: 100%;

  &__speak-amount {
    margin-left: 30px;
  }
}
</style>
