<template>
  <section class="card-dates-edit">
    <div class="title">
      <span>{{ action.name }}</span>
      <span class="close material-icons close-popup-btn" @click.stop="close"
        >close</span
      >
    </div>
    <main>
      <date-picker
        v-model="cardToEdit.dueDate.time"
        :inline="true"
        :default-value="new Date()"
        valueType="format"
      >
      </date-picker>
      <el-button
        type="primary"
        class="save"
        :disabled="!cardToEdit.dueDate.time"
        @click="updateCard"
        >Save</el-button
      >
    </main>
  </section>
</template>

<script>
import { eventBus } from "@/services/event-bus-service";
import DatePicker from "vue2-datepicker";
import "vue2-datepicker/index.css";
export default {
  props: {
    action: {
      type: Object,
      required: true,
    },
    card: {
      type: Object,
      required: true,
    },
  },
  data() {
    return {
      cardToEdit: null,
    };
  },
  components: {
    DatePicker,
  },
  created() {
    this.cardToEdit = JSON.parse(JSON.stringify(this.card));
  },
  methods: {
    updateCard() {
      this.cardToEdit.dueDate.isDone = false;
      const activity = {
        txt: `set ${this.card.title} to be due ${this.cardToEdit.dueDate.time}`,
        card: { id: this.card.id, title: this.card.title },
      };
      console.log(' this.cardToEdit',  this.cardToEdit)
      this.$emit("updateCard", this.cardToEdit, true);
      eventBus.$emit("addActivity", activity);
      this.close();
    },
    close() {
      this.$emit("close");
    },
  },
};
</script>
