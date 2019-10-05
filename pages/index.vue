<template>
  <div class="container">
    <div>
      <img id="logo" src="/logo.svg">
      <p class="title">
        do you know your
        <select v-on:change="changeSetting" v-model="setting">
          <option value="html5">HTML 5</option>
          <option value="html4">HTML 4</option>
          <option value="html2">HTML 2</option>
          <option value="html0">HTML 0.d</option>
        </select> tags?
      </p>

      <p>
        Rules: you've got {{ count }} seconds, each time you get a correct tag, you gain
        a second.
      </p>

      <main>
        <div class="flex flex__2">
          <form v-on:submit.prevent="submit">
            <input :disabled="over" id="tag" autofocus>
          </form>
          <div class="progress">{{ found.length }} / {{ count }}</div>
        </div>

        <Progress :value="progress" :max="count"/>

        <ul class="found">
          <li :key="tag" v-for="tag in found"><a :href="tag | toMDN">{{ tag }}</a></li>
          <li v-show="progress === count && !showRemainder">
            <button v-on:click="show">See what you missed?</button>
          </li>
          <li class="missed" v-show="showRemainder" :key="tag" v-for="tag in remainder"><a :href="tag | toMDN">{{ tag }}</a></li>
        </ul>
      </main>
    </div>
    <script async defer crossorigin="anonymous" src="https://api.countapi.xyz/hit/tags.isthe.link?callback=cb"></script>
  </div>
</template>

<script>
import Progress from '~/components/Progress';
import allCorpus from '~/lib/corpus';

let timer = null;

export default {
  components: {
    Progress,
  },
  mounted() {
    this.start();
  },
  filters: {
    toMDN(tag) {
      return 'https://developer.mozilla.org/en-US/docs/Web/HTML/Element/' + tag;
    }
  },
  methods: {
    changeSetting($event) {
      this.corpus = allCorpus[this.setting];
      window.history.replaceState(null, null, '?setting=' + this.setting);
      this.reset();
      this.start();
    },
    reset() {
      this.over = false;
      this.found = [];
      this.showRemainder = false;
      this.remainder = Array.from(this.corpus);
      this.count = this.corpus.length;
      this.progress = 0;
    },
    start() {
      clearInterval(timer);
      timer = setInterval(() => {
        this.progress++;
        if (this.progress === this.corpus.length) {
          this.over = true;

          clearInterval(timer);
        }
      }, 1000);
    },
    show() {
      this.showRemainder = true;
    },
    submit($event) {
      const tag = $event.target.tag.value.toLowerCase().trim();
      $event.target.tag.value = '';
      if (this.found.includes(tag)) {
        return;
      }

      if (this.corpus.includes(tag)) {
        this.found.push(tag);
        this.remainder = this.corpus.filter(_ => !this.found.includes(_));
        this.progress--;
      }
    },
  },
  data() {
    let setting = 'html5';

    if (typeof location !== 'undefined' && location.search) {
      const match = location.search.match(/setting=(.*)\b/);
      if (match) {
        if (allCorpus[match[1]]) {
          setting = match[1];
        }
      }

    }
    const corpus = allCorpus[setting];
    return {
      setting,
      over: false,
      found: [],
      showRemainder: false,
      corpus,
      remainder: Array.from(corpus),
      count: corpus.length,
      progress: 0,
    };
  },
};
</script>

<style scoped>
button, a {
  background: none;
  border: 0;
  font-size: inherit;
  font-family: inherit;
  color: rgb(0, 132, 74);
  text-decoration: underline;
  cursor: pointer;
}

.flex {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  margin: 40px 0;
}

.flex > * {
  flex: 1;
}

.progress {
  font-family: 'ubuntu mono', monospace;
  font-size: 2rem;
  font-weight: bold;
  margin-left: 20px;
}

.missed a {
  color: rgb(132, 0, 74);
}

input {
  margin-right: 20px;
  font-family: 'ubuntu mono', monospace;
  font-size: 2rem;
  padding: 8px;
  border: 1px solid #ccc;
  text-align: center;
  width: 100%;
}

#logo {
  max-width: 200px;
}

main {
  margin: 0 auto;
  max-width: 90vw;
}

.container {
  margin: 20px auto;
  display: flex;
  justify-content: center;
  text-align: center;
  position: relative;
}

.container > div {
  width: 80vw;
  max-width: 800px;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 200%;
  line-height: 3rem;
  color: #35495e;
  letter-spacing: 1px;
  margin: 40px 0;
}

.found {
  max-width: 80vw;
  padding: 0;
  margin: 0 auto;
  display: flex;
  flex-wrap: wrap;
  list-style: none;
}

.found > * {
  margin: 8px;
}

select {
  font-size: 90%;
}
</style>
