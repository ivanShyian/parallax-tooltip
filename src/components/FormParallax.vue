<template>
  <div class="FormParallax w-full flex justify-between items-start relative gap-x-20">
    <div
      class="flex-1 flex items-start justify-end relative "
      :style="{height: `${wrapperHeight}px`}"
    >
      <span
        class="rounded-[14px] p-4 mt-[168px] flex bg-white sticky duration-300 text-lg ease-linear"
        :style="hintTopProperty"
        :class="[ isTransitionAllow ? 'transition-all' : 'transition-none' ]"
      >
        {{ form[activeHintIndex].hint }}</span>
    </div>
    <div
      id="formWrapper"
      class="bg-gray-200 w-1/3 rounded-[10px] p-10 overflow-hidden sticky"
      :style="formTopProperty"
    >
      <form
        action="/foo"
        method="post"
        class="flex flex-wrap py-4 gap-4"
      >
        <p class="font-bold text-3xl rotate-2"><span class="pl-20 block transform rotate-6">LOCKDOWN</span></p>
        <template
          v-for="field in form"
          :key="field.name"
        >
          <label
            :for="field.name"
            class="flex w-full flex-col"
            :class="{ '!w-[calc(50%-8px)]': field.isHalf }"
          >
            {{ field.label }}
            <input
              type="text"
              :id="field.name"
              class="FormInput inp"
            >
          </label>
        </template>
        <button class="mt-10 border border-black">SEND THIS DAMN FORM</button>
      </form>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import _throttle from 'lodash.throttle'

const FORM: {
  name: string;
  label: string;
  placeholder: string;
  hint: string;
  isHalf?: boolean;
}[] = [
  {
    name: 'name',
    label: 'Name',
    placeholder: 'Fill out ur name',
    hint: 'There is name of your name man',
  },
  {
    name: 'name1',
    label: 'Name of your mom',
    placeholder: 'Fill out ur mom name',
    hint: 'is name of your name man',
  },
  {
    name: 'name2',
    label: 'Name of your mom',
    placeholder: 'Fill out ur mom name',
    hint: 'name of your name man',
  },
  {
    name: 'name3',
    label: 'Name of',
    placeholder: 'Fill out ur mom name',
    hint: 'of your name man',
    isHalf: true,
  },
  {
    name: 'name4',
    label: 'your mom',
    placeholder: 'Fill out ur mom name',
    hint: 'your name man',
    isHalf: true,
  },
  {
    name: 'name5',
    label: 'Name of your mom',
    placeholder: 'Fill out ur mom name',
    hint: 'name man',
  },
];

enum EventListenerActions {
  ADD = 'addEventListener',
  REMOVE = 'removeEventListener'
}

const SCROLL_STEP_HEIGHT = 1000;

/*
* FIXME If scroll start from end of the page hint starts move
*  FIXME without recognizing that scroll going from end
* */

export default defineComponent({
  data: () => ({
    form: FORM,

    activeHintIndex: 0,
    formHeight: 0,
    wrapperHeight: 0,
    fieldsHeight: [] as { top: number; left: number }[],
    isTransitionAllow: false,
  }),

  computed: {
    formTopProperty(): Record<'top', string> {
      return { top: `calc(50% - ${this.formHeight / 2}px)` };
    },
    hintTopProperty(): Record<string, string> {
      // FIXME Hint position needs to be normalized programmatically instead of static value (CALIBRATION): src/components/FormParallax.vue:128
      // FIXME Create a constant for 50 value: src/components/FormParallax.vue:134
      if (this.fieldsHeight.length && this.formHeight) {
        const activeHintIndex = this.fieldsHeight[this.activeHintIndex];

        /* top + marginTop is related to normalize hint position */
        const CALIBRATION = 10;
        const top = `calc(50% - ${this.formHeight / 2}px + ${activeHintIndex.top - CALIBRATION}px)`;
        const marginTop = `${this.fieldsHeight[0].top - CALIBRATION}px`;

        const marginRight = `-${activeHintIndex.left}px`;
        const marginBottom = `${this.formHeight - this.fieldsHeight.at(-1)!.top - 50}px`;

        return {
          top,
          marginRight,
          marginTop,
          marginBottom,
        }
      }
      return {};
    },
  },

  methods: {
    setWrapperHeight(wrapper: HTMLDivElement) {
      this.wrapperHeight = SCROLL_STEP_HEIGHT * this.form.length + wrapper.clientHeight;
    },
    setFormHeight(wrapper: HTMLDivElement) {
      this.formHeight = wrapper.clientHeight;
    },
    setFieldsHeight() {
      // Save the every input top property relative to form
      const inputs = document.querySelectorAll('.FormInput') as NodeListOf<HTMLInputElement>;
      Array.from(inputs).forEach((input) => {
        const offsetTop = input.offsetTop as number;
        const position = {
          top: offsetTop,
          left: 0,
        }
        // Can be removed. Sets the left to current value if previous value same as current
        if (this.fieldsHeight.length && this.fieldsHeight.at(-1)!.top === position.top) {
          position.left = 20
        }
        this.fieldsHeight.push(position)
      });
    },

    scrollCallback(event: Event, wrapper: HTMLElement) {
      const documentElementHeight = document.documentElement.clientHeight;
      // Top and bottom margins are equal because of src/components/FormParallax.vue:130
      const wrapperMargins = wrapper.getBoundingClientRect().top * 2;
      // ComputedStyle that gives us result of src/components/FormParallax.vue:130
      const calculatedFormTop = parseInt(window.getComputedStyle(wrapper, null).getPropertyValue("top"));
      // True if form being sticky
      const isStraightCentered = (documentElementHeight - wrapper.clientHeight - wrapperMargins) === 0;
      const scrolledHeight = (document.querySelector('.FormParallax') as HTMLDivElement).getBoundingClientRect().top;
      if (isStraightCentered) {
        this.activeHintIndex = Math.floor(Math.abs(scrolledHeight - calculatedFormTop) / SCROLL_STEP_HEIGHT);
      }
    },

    setScrollListener(
      wrapper: HTMLElement,
      action: EventListenerActions = EventListenerActions.ADD,
    ) {
      document[action]('scroll', _throttle((event) => this.scrollCallback(event, wrapper), 300));
    },

    getFormWrapper(): HTMLElement | null {
      return document.getElementById('formWrapper');
    },
  },

  mounted() {
    const wrapper = document.getElementById('formWrapper') as HTMLDivElement;
    this.setWrapperHeight(wrapper);
    this.setFormHeight(wrapper);
    this.setFieldsHeight();
    this.setScrollListener(wrapper);

    // Enables transition
    setTimeout(() => {
      this.isTransitionAllow = true;
    }, 300)
  },

  beforeUnmount() {
    const wrapper = this.getFormWrapper();
    if (wrapper) {
      this.setScrollListener(wrapper, EventListenerActions.REMOVE);
    }
  }
});
</script>

<style scoped>
.inp {
  @apply h-10 w-full border border-yellow-400 rounded-[10px];
}
</style>