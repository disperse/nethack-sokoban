<template>
  <div id="app">
    <h1 v-if="!won">Sokoban level {{ (this.curLevel + 1) + ((this.curSubLevel === 0) ? 'a' : 'b')}}</h1>
    <Map v-if="!won" :map-string="mapString" :player-position="playerPosition"/>
    <h3 v-if="!won">Push R to restart.</h3>
    <h1 v-if="won">YOU WIN!</h1>
    <div>
      <h3>Options</h3>
      <input type="checkbox" v-model="doBothSubLevels" id="doBothSubLevels"><label for="doBothSubLevels">Do both sub-levels</label>
    </div>
    <div>
      <h3>Levels</h3>
      <ul>
        <li v-for="level in levelListing" v-bind:key="level.name"><a href="#" @click="changeLevel(level)">{{level.name}}</a></li>
      </ul>
    </div>
  </div>
</template>

<script>
import Map from './components/Map.vue'
import maps from './maps/maps'

export default {
  name: 'App',
  components: {
    Map
  },
  created () {
    window.addEventListener('keypress', this.keyboardEvent);
  },
  destroyed () {
    window.removeEventListener('keypress', this.keyboardEvent);
  },
  mounted () {
    this.curSubLevel = (this.doBothSubLevels) ? 0 : Math.floor(Math.random() * maps[this.curLevel].length)
    this.loadMap();
  },
  data () {
    return {
      mapString: '',
      curLevel: 0,
      curSubLevel: 0,
      doBothSubLevels: false,
      flipLevels: false, // TODO
      map: [],
      playerPosition: [-1, -1],
      won: false
    }
  },
  computed: {
    levelListing () {
      const listing = [];
      for (let i = 0; i < maps.length; i++) {
        for (let j = 0; j < maps[i].length; j++) {
          listing.push({
            name: 'Sokoban Level ' + (i + 1) + ((j === 0) ? 'a' : 'b'),
            level: i,
            subLevel: j
          })
        }
      }
      return listing;
    }
  },
  methods: {
    changeLevel (levelObject) {
      this.curLevel = levelObject.level;
      this.curSubLevel = levelObject.subLevel;
      this.loadMap();
    },
    updateMapString () {
      let mapString = '';
      for (let y = 0; y < this.map.length; y++) {
        mapString += (this.playerPosition[1] === y) ?
            this.map[y].substr(0, this.playerPosition[0]) + '@' + this.map[y].substr(this.playerPosition[0] + 1) + '\n'
            : this.map[y] + '\n';
      }
      this.mapString = mapString;
    },
    loadMap () {
      this.map = maps[this.curLevel][this.curSubLevel].slice();
      this.playerPosition = this.getPlayerPosition(this.map);
      this.updateMapString();
    },
    getPlayerPosition (map) {
      for (let y = 0; y < map.length; y++) {
        if (map[y].indexOf('>') > 0) {
          return [(map[y].indexOf('>')), y];
        }
      }
      return [-1, -1];
    },
    charAt (map, x, y) {
      return map[y].charAt(x);
    },
    keyboardEvent (evt) {
      if (evt.code === 'KeyJ' || evt.code === 'Numpad2') this.move(0, 1);
      if (evt.code === 'KeyK' || evt.code === 'Numpad8') this.move(0, -1);
      if (evt.code === 'KeyH' || evt.code === 'Numpad4') this.move(-1, 0);
      if (evt.code === 'KeyL' || evt.code === 'Numpad6') this.move(1, 0);
      if (evt.code === 'KeyY' || evt.code === 'Numpad7') this.move(-1, -1);
      if (evt.code === 'KeyU' || evt.code === 'Numpad9') this.move(1, -1);
      if (evt.code === 'KeyB' || evt.code === 'Numpad1') this.move(-1, 1);
      if (evt.code === 'KeyN' || evt.code === 'Numpad3') this.move(1, 1);
      if (evt.code === 'KeyR') this.loadMap();
    },
    move (x, y) {
      if (this.canMoveTo(this.playerPosition[0] + x, this.playerPosition[1] + y, x, y, false)) {
        this.playerPosition[0] += x;
        this.playerPosition[1] += y;
        this.updateMapString();
      }
    },
    isPassable(x, y) {
      const movingTo = this.charAt(this.map, x, y);
      return (movingTo === '.' || movingTo === '<' || movingTo === '>' || movingTo === '+');
    },
    canMoveTo (x, y, dx, dy) {
      const movingTo = this.charAt(this.map, x, y)
      // Prevent squeezing through
      if (dx !== 0 && dy !== 0) {
        // if moving diagonal
        if (!this.isPassable((x - dx), y) && !this.isPassable(x, (y - dy))) {
          return false
        }
      }
      if (this.isPassable(x, y)) {
        if (movingTo === '<') {
          if (this.curLevel === maps.length - 1 && (!this.doBothSubLevels || this.curSubLevel === maps[this.curLevel].length - 1)) {
            this.won = true;
          } else {
            if (this.doBothSubLevels) {
              if (this.curSubLevel === maps[this.curLevel].length - 1) {
                this.curLevel++;
                this.curSubLevel = 0;
              }
            } else {
              this.curLevel++;
              this.curSubLevel = Math.floor(Math.random() * maps[this.curLevel].length)
            }
            this.loadMap();
          }
          return false
        } else {
          return true
        }
      } else if (movingTo === '0') {
        return this.tryPushBoulder(x, y, dx, dy)
      }
      return false
    },
    tryPushBoulder (x, y, dx, dy) {
      // can't push boulders diagonal
      if (dx !== 0 && dy !== 0) return false;
      const pushingTo = this.charAt(this.map, x + dx, y + dy);
      if (pushingTo === '.' || pushingTo === '<' || pushingTo === '>') {
        // Move boulder
        this.map[y + dy] = this.map[y + dy].substr(0, x + dx) + '0' + this.map[y + dy].substr(x + dx + 1);
      } else if (pushingTo === '^') {
        // Replace pit with floor
        this.map[y + dy] = this.map[y + dy].substr(0, x + dx) + '.' + this.map[y + dy].substr(x + dx + 1);
      } else {
        return false;
      }
      // Replace boulder's previous position with the initial map tile or floor
      const prevPosition = this.charAt(maps[this.curLevel][this.curSubLevel], x, y);
      if (prevPosition === '0' || prevPosition === '.' || prevPosition === '^') {
        this.map[y] = this.map[y].substr(0, x) + '.' + this.map[y].substr(x + 1);
      } else {
        // This prevents stairs from being deleted when a boulder is moved on them
        this.map[y] = this.map[y].substr(0, x) + this.charAt(maps[this.curLevel][this.curSubLevel], x, y) + this.map[y].substr(x + 1);
      }
      return true;
    }
  },
  watch: {
    doBothSubLevels: function (val) {
      if (val && this.curSubLevel > 0) {
        this.curSubLevel = 0;
        this.loadMap();
      }
    }
  }
}
</script>

<style>
</style>
