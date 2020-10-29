<style>
body {
  background-color: #ccc;
}
.info {
  display: inline-grid;
  padding: 4px;
}
#filters {
  margin: 20px 0 10px 0;
}
#filters .btn {
  margin-right: 5px;
  font-family: "HelveticaNeueLTStd-Lt";
  padding-top: 10px;
  margin-bottom: 10px;
}
#filters .btn-primary,
#filters .btn-default {
  transition: all 0.2s ease 0.2s;
}
.btn {
  box-shadow: 1px 1px 4px #808080;
}
.btn-default,
.btn-default:hover {
  background-color: #000;
  border-color: #000;
  color: #fff;
}
.grid-item-content:hover {
  border-bottom: 4px solid #239AD8;
}
.btn-primary,
.btn-primary:hover {
  background-color: #239AD8;
  border-color: #239AD8;
}
.grid > .row {
  margin-bottom: 20px;
}
.grid-item {
  height: 240px;
  display: inline-grid;
  margin-bottom: 20px;
}
.grid-item a {
  display: grid;
}
.grid-item .well {
  height: 100%;
  width: 100%;
  padding: 20px;
}
.grid-item-content {
  height: 100%;
  background-color: #fff;
  padding: 0 20px 20px;
}
#realizations_entries .grid-item-content {
  filter: grayscale(100%);
  box-shadow: 5px 5px 5px #ccc;
}
#realizations_entries .grid-item-content:hover {
  filter: none;
}
#realizations_entries .grid-item-content:hover a {
  text-decoration: none;
}
#realizations_entries .grid-item-content:hover h3 {
  color: #249FDE;
}
.grid-sizer {
  display: none;
}
</style>

<template>
  <div :title="'Portfolio'">
    <section v-if="items" id="realizations_index" class="greyBG">
      <div class="row">
        <div class="col-lg-12">
          <div id="filters" class="button-group">
            <button class="button btn btn-primary" @click="filter('*')" data-filter="*">
              Tous
              <small>({{ items.length }})</small>
            </button>
            
            <button
              v-for="item in items"
              :key="item.slug"
              @click="filter('.' + item.slug)"
              :class="'button btn btn-default ' + item.slug"
              :data-filter="item.slug"
            >{{ item.first_name }} ({{ occurrences(item.slug) }})</button>
          </div>
        </div>
      </div>

      <div class="row">
        <div class="col-lg-12">
          <div id="realizations_entries" class="row grid isotope">
            <div class="grid-sizer col-lg-3"></div>

            <div v-for="item in items" :key="item.id" :class="'col-lg-3 grid-item ' + item.slug">
              <div class="grid-item-content">
                <nuxt-link
                  :to="'/portfolio/' + item.id"
                  :title="item.first_name + ' ' + item.last_name"
                >
                  <h3 class="ellipsis">{{ item.first_name }}</h3>
                </nuxt-link>

                <nuxt-link
                  :to="'/portfolio/' + item.id"
                  :title="item.first_name + ' ' + item.last_name"
                >
                  <img
                    :title="item.first_name + ' ' + item.last_name"
                    :alt="item.first_name + ' ' + item.last_name"
                    :src="item.avatar.src"
                  >
                  <div class="info">
                    <h4 class="ellipsis" v-html="item.first_name"></h4>
                  </div>
                </nuxt-link>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
let Isotope;

if (process.browser) {
  Isotope = require("isotope-layout");
}

export default {
  data() {
    return {
      iso: null,
      items: [],
      itemsOccurences: {}
    };
  },

  async asyncData({ $axios }) {
    let itemsOccurences = {};
    let { data } = await $axios.get(`https://reqres.in/api/users?per_page=30`);

    data.data.forEach(item => {
      let tmp = item.last_name.replace(/ /g, "-").replace(/\./, "_");

      if (typeof itemsOccurences[tmp] === "undefined") {
        itemsOccurences[tmp] = 1;
        item["slug"] = tmp;
      } else {
        itemsOccurences[tmp]++;
      }
    });

    return {
      items: data.data,
      itemsOccurences: itemsOccurences
    };
  },

  mounted() {
    this.isotope();
  },

  methods: {
    isotope() {
      this.iso = new Isotope(".grid", {
        itemSelector: ".grid-item",
        percentPosition: true,
        masonry: {
          columnWidth: ".grid-sizer"
        }
      });

      this.iso.layout();
    },
    formatSlug: function(data) {
      return data ? data.replace(/ /g, "-").replace(/\./, "_") : "";
    },
    occurrences: function(slug) {
      return this.itemsOccurences[slug];
    },
    filter: function(slug) {
      let oldActive = $("#filters .btn-primary").first();

      if (slug === oldActive.data("filter")) {
        return;
      }

      let currentActive = $("#filters button" + slug).first();

      currentActive.removeClass("btn-default").addClass("btn-primary");

      oldActive.removeClass("btn-primary").addClass("btn-default");

      this.iso.arrange({
        filter: slug
      });
    }
  }
};
</script>
