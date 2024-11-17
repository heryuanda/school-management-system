<template>
  <div class="container mx-auto p-4">
    <div class="grid place-content-center min-h-screen">
      <div class="grid gap-4 grid-cols-2">
        <div class="bg-[#f5f5f5] rounded-lg p-6">
          <h2 class="text-2xl font-bold mb-4 text-neutral-600">
            Settings Form
          </h2>
          <div class="space-y-4">
            <div class="grid grid-cols-2 gap-4">
              <div>
                <label
                  for="visit-duration"
                  class="text-sm text-neutral-700 font-medium mb-1 inline-block pl-3"
                  >Visit Duration</label
                >
                <select
                  id="visit-duration"
                  v-model="visitDuration"
                  class="px-3 py-3 focus:outline-none border border-neutral-300 w-full rounded-lg"
                  placeholder="select"
                >
                  <option value="" disabled>Select</option>
                  <option v-for="duration in DURATION" :value="duration">
                    {{ duration + " min" }}
                  </option>
                </select>
              </div>
              <div>
                <InputField
                  label="No. of Booking / Session"
                  type="number"
                  v-model="totalSession"
                />
              </div>
            </div>
            <div class="flex items-center space-x-2">
              <input
                id="allow-video-tour"
                v-model="allowVideoTour"
                type="checkbox"
                class="rounded"
              />
              <label
                for="allow-video-tour"
                class="text-sm font-medium text-neutral-600"
                >Allow video tour call</label
              >
            </div>
            <div class="p-4 bg-white rounded-md border border-neutral-300">
              <h3 class="text-lg font-bold text-neutral-600">Availability</h3>
              <p class="mb-4 text-sm text-neutral-600">
                Set your weekly recurring schedule
              </p>
              <div
                v-for="(day, dayIndex) in DAYS"
                :key="day"
                class="pb-3 mb-3 border-neutral-300 border-b last:border-none last:mb-0"
              >
                <div
                  class="flex space-x-4"
                  :class="[
                    enabledDays[dayIndex] ? 'items-start' : 'items-center',
                  ]"
                >
                  <div
                    class="flex space-x-4 w-20 items-center leading-[0]"
                    :class="[enabledDays[dayIndex] ? 'pt-4' : 'pt-0']"
                  >
                    <input
                      :id="`day-${day}`"
                      v-model="enabledDays[dayIndex]"
                      type="checkbox"
                      class="rounded"
                      @change="handleToggle(dayIndex)"
                    />
                    <p
                      :for="`day-${day}`"
                      class="text-sm font-bold text-neutral-600"
                    >
                      {{ day }}
                    </p>
                  </div>
                  <template v-if="!enabledDays[dayIndex]">
                    <p class="text-sm text-neutral-400 pl-3">Unavailable</p>
                  </template>
                  <template v-else>
                    <div class="w-full">
                      <div
                        v-for="(slot, slotIndex) in timeSlots[dayIndex]"
                        :key="slotIndex"
                        class="flex items-center pl-4 mb-2 gap-x-2"
                      >
                        <select
                          v-model="slot.startTime"
                          class="px-3 py-3 focus:outline-none border border-neutral-300 w-full rounded-lg"
                          @change="updateCalendar"
                        >
                          <option
                            v-for="hour in 13"
                            :key="hour"
                            :value="`${(hour + 6)
                              .toString()
                              .padStart(2, '0')}:00`"
                          >
                            {{
                              `${hour + 6}:00 ${hour + 6 < 12 ? "AM" : "PM"}`
                            }}
                          </option>
                        </select>
                        <div>-</div>
                        <div
                          class="px-3 py-3 focus:outline-none border border-neutral-300 w-full rounded-lg bg-neutral-100 text-neutral-400"
                        >
                          {{ formattedEndTime(slot.startTime) }}
                        </div>
                        <button
                          @click="removeBookedSlot(dayIndex, slotIndex)"
                          class="p-2 text-neutral-400 disabled:cursor-not-allowed"
                        >
                          <Trash2Icon class="h-5 w-5" />
                        </button>
                      </div>
                    </div>
                  </template>
                </div>
                <div class="flex justify-end">
                  <button
                    v-if="enabledDays[dayIndex]"
                    @click="addTimeSlot(dayIndex)"
                    class="mt-2 inline-flex items-center px-3 py-2 border border-neutral-300 text-sm rounded-md text-neutral-600 bg-white"
                  >
                    <PlusIcon class="h-4 w-4 mr-2" />
                    Add More
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div class="bg-[#f5f5f5] rounded-lg p-6">
          <h2 class="text-2xl font-bold mb-4">Scheduler Table</h2>
          <vue-cal
            style="height: 600px"
            :events="eventsList"
            selected-date="2018-11-19"
            :disable-views="['years', 'year', 'month']"
            :time-from="7 * 60"
            :time-to="20 * 60"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, watch, defineAsyncComponent } from "vue";
import VueCal from "vue-cal";
import dayjs from "dayjs";
import "vue-cal/dist/vuecal.css";
import { Trash2Icon, PlusIcon } from "lucide-vue-next";
import { TimeSlot, Event } from "./types";

const InputField = defineAsyncComponent(
  () => import("./components/form/Input.vue")
);

const DAYS = ["Mon", "Tue", "Wed", "Thu", "Fri", "Sat", "Sun"];
const DURATION = [15, 30, 45, 60, 90];

const visitDuration = ref<number>(30);
const totalSession = ref<number>(1);
const allowVideoTour = ref<boolean>(false);
const enabledDays = ref<[]>(DAYS.map(() => false));
const timeSlots = ref<TimeSlot[][]>(DAYS.map(() => []));
const eventsList = ref<Event[]>([]);

const addTimeSlot = (dayIndex: number): void => {
  timeSlots.value[dayIndex].push({ startTime: "07:00" });
  updateCalendar();
};

const removeBookedSlot = (dayIndex: number, slotIndex: number): void => {
  timeSlots.value[dayIndex].splice(slotIndex, 1);
  if (!timeSlots.value[dayIndex].length) {
    enabledDays.value[dayIndex] = false;
  }
  updateCalendar();
};

const handleToggle = (dayIndex: number): void => {
  if (enabledDays.value[dayIndex]) {
    timeSlots.value[dayIndex] = [{ startTime: "07:00" }];
  } else {
    timeSlots.value[dayIndex] = [];
    eventsList.value = eventsList.value.filter(
      (event) => event.dayOfWeek !== dayIndex
    );
  }
  updateCalendar();
};

const formattedEndTime = (startTime: string): string => {
  const [hours, minutes] = startTime.split(":").map(Number);
  const endDate = new Date();
  endDate.setHours(hours, minutes, 0, 0);
  endDate.setMinutes(endDate.getMinutes() + visitDuration.value);
  return `${endDate.getHours().toString().padStart(2, "0")}:${endDate
    .getMinutes()
    .toString()
    .padStart(2, "0")}`;
};

const updateCalendar = () => {
  eventsList.value = [];

  timeSlots.value.forEach((daySlots, dayIndex) => {
    daySlots.forEach((slot) => {
      const [hours, minutes] = slot.startTime.split(":").map(Number);
      const startTime = new Date("2018-11-19");
      startTime.setDate(startTime.getDate() + dayIndex);
      startTime.setHours(hours, minutes, 0, 0);
      const endTime = new Date(
        startTime.getTime() + visitDuration.value * 60000
      );

      for (let i = 0; i < totalSession.value; i++) {
        eventsList.value.push({
          start: dayjs(startTime).format("YYYY-MM-DD HH:mm"),
          end: dayjs(endTime).format("YYYY-MM-DD HH:mm"),
          class: "booked-slot",
          title: `Booking Slot from ${dayjs(startTime).format(
            "HH:mm"
          )} to ${dayjs(endTime).format("HH:mm")}`,
          dayOfWeek: dayIndex,
        });
      }
    });
  });
};

watch([visitDuration, totalSession], updateCalendar);
</script>

<style lang="css">
.booked-slot {
  background-color: #4169e1;
  border: 1px solid;
  font-size: 12px;
  color: #fff;
}
</style>
