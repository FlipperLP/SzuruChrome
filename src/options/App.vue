<template>
  <div class="content-holder">
    <div class="content-wrapper">
      <strong>SzuruChrome Settings</strong>

      <p>General settings which apply to all engines.</p>

      <ul class="input">
        <li class="full-width">
          <label>Szurubooru URL</label><input text="Szurubooru URL" type="text" name="domain" v-model="domain" />
        </li>
        <li><label>Username</label><input text="Username" type="text" name="username" v-model="username" /></li>
        <li>
          <label>Authentication token</label
          ><input text="Authentication token" type="text" name="token" v-model="authToken" />
        </li>
      </ul>

      <ul class="input">
        <li>
          <label>
            <input type="checkbox" v-model="autoSearchSimilar" />
            Automatically search for similar posts
          </label>
          <p class="hint">
            Automatically start searching for similar posts when opening the popup. Enabling this option will hide the
            "Find Similar" button.
          </p>
        </li>
        <li>
          <label>
            <input type="checkbox" v-model="useOriginalSource" />
            Use original image source when available
          </label>
          <p class="hint">
            Use the original image source when available. Usually this will mean a Twitter, DeviantArt, ArtStation, or
            Pixiv page.
          </p>
        </li>
        <li>
          <label>
            <input type="checkbox" v-model="loadTagCounts" />
            Show how often a tag is used in the selected szurubooru instance
          </label>
          <p class="hint">
            Shows how often a given tag is used in the selected szurubooru instance. No number will be displayed when
            the tag does not yet exist.
          </p>
        </li>
        <li>
          <label>
            <input type="checkbox" v-model="loadTagCounts" />
            Use temporary files when reverse searching and creating posts
          </label>
          <p class="hint">
            When reverse searching for a post this will (temporarily) upload the post to the server. This makes creating
            the post faster because then the file already exists on the server.
          </p>
        </li>
      </ul>

      <div class="settings-footer">
        <span class="status" :class="statusType">{{ statusText }}</span>

        <button @click="testConnection">Test connection</button>
        <button @click="saveSettings" class="primary">Save settings</button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import SzuruWrapper from "../SzuruWrapper";
import { Config, SzuruSiteConfig } from "../Config";

type StatusType = "ok" | "error";

export default Vue.extend({
  data() {
    return {
      domain: "",
      username: "",
      authToken: "",
      autoSearchSimilar: false,
      loadTagCounts: false,
      useOriginalSource: false,
      useContentTokens: false,
      statusText: "",
      statusType: "",
    };
  },
  methods: {
    async testConnection() {
      if (!this.domain || !this.username || !this.authToken) {
        this.setStatus("Domain, username and authentication token are all required.", "error");
        return;
      }

      const api = new SzuruWrapper(this.domain, this.username, this.authToken);
      try {
        const info = (await api.getInfo()) as any;
        const instanceName = info?.config.name;

        if (instanceName == undefined) {
          this.setStatus(`Connected to ${this.domain}, but it is not a szurubooru instance.`, "error");
        } else {
          this.setStatus(`Connected to ${info.config.name} at ${this.domain}`);
        }
      } catch (ex) {
        // TODO: This error message is not very descriptive.
        console.dir(ex);
        this.setStatus("Couldn't connect to " + this.domain + "\n\n" + ex, "error");
      }
    },
    async saveSettings() {
      let config = await Config.load();

      // We currently only support one domain
      let siteConfig = new SzuruSiteConfig();
      siteConfig.domain = this.domain;
      siteConfig.username = this.username;
      siteConfig.authToken = this.authToken;
      config.sites = [siteConfig];
      config.autoSearchSimilar = this.autoSearchSimilar;
      config.loadTagCounts = this.loadTagCounts;
      config.useOriginalSource = this.useOriginalSource;
      config.useContentTokens = this.useContentTokens;

      await config.save();

      this.setStatus("Settings successfully saved");
    },
    setStatus(text: string, type: StatusType = "ok") {
      this.statusText = text;
      this.statusType = "status-" + type;
    },
  },
  async mounted() {
    let config = await Config.load();

    // console.log(`Loaded ${config.sites.length} sites`);

    // We currently only support one domain
    if (config.sites.length > 0) {
      this.domain = config.sites[0].domain;
      this.username = config.sites[0].username;
      this.authToken = config.sites[0].authToken;
    }

    this.autoSearchSimilar = config.autoSearchSimilar;
    this.loadTagCounts = config.loadTagCounts;
    this.useOriginalSource = config.useOriginalSource;
    this.useContentTokens = config.useContentTokens;
  },
});
</script>

<style scoped>
.content-holder {
  padding: 1.5em;
  display: flex;
  justify-content: center;
}

.content-holder > .content-wrapper {
  box-sizing: border-box;
  text-align: left;
  display: inline-block;
  flex: auto;
  max-width: 1000px;
}

.content-holder > .content-wrapper:not(.transparent) {
  background: #f5f5f5;
  padding: 1.8em;
}

.settings-footer {
  display: flex;
  gap: 5px;
  flex-wrap: wrap;
}

.settings-footer .status {
  flex-grow: 1;
}

.settings-footer > span.status-ok {
  color: green;
}

.settings-footer > span.status-error {
  color: red;
}

.input {
  list-style-type: none;
  margin: 0 0 2em;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.input li {
  flex: 1 0 20em;
}

.input li.full-width {
  min-width: 100%;
}

.input label {
  display: block;
  padding: 0.3em 0;
}

.hint {
  margin-top: 0.2em;
  margin-bottom: 0;
  color: #888;
  font-size: 80%;
  line-height: 120%;
}

input[type="email"],
input[type="number"],
input[type="password"],
input[type="text"],
select,
textarea {
  vertical-align: top;
  font-family: "Droid Sans", sans-serif;
  font-size: 100%;
  padding: 0.2em 0.3em;
  text-overflow: ellipsis;
  width: 100%;
  box-sizing: border-box;
  border: 2px solid #eee;
  background: #fafafa;
  color: #111;
  box-shadow: none;
  transition: border-color 0.1s linear, background-color 0.1s linear;
}
</style>
