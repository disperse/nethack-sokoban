<template>
  <div id="app">
    <h1 v-if="!won">Sokoban level {{ (this.curLevel + 1) + ((this.curSubLevel === 0) ? 'a' : 'b')}}</h1>
    <Map v-if="!won" :map-string="mapString" :player-position="playerPosition"/>
    <h3 v-if="!won">Push R to restart.</h3>
    <h1 v-if="won">YOU WIN!</h1>
    <div>
      <h3>Options</h3>
      <div>
        <input type="checkbox" v-model="doBothSubLevels" id="doBothSubLevels"><label for="doBothSubLevels">Do both sub-levels</label>
      </div>
      <div>
        <input type="checkbox" v-model="flipHorizontally" id="flipHorizontally"><label for="flipHorizontally">Flip levels horizontally</label>
      </div>
      <div>
        <input type="checkbox" v-model="flipVertically" id="flipVertically"><label for="flipVertically">Flip levels vertically</label>
      </div>
      <div>
        <input type="checkbox" v-model="flipRandomly" id="flipRandomly"><label for="flipRandomly">Flip levels randomly</label>
      </div>
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
    window.addEventListener('keydown', this.keyboardEvent);
  },
  destroyed () {
    window.removeEventListener('keydown', this.keyboardEvent);
  },
  mounted () {
    this.curSubLevel = (this.doBothSubLevels) ? 0 : Math.floor(Math.random() * maps[this.curLevel].length)
    this.loadMap();
  },
  data () {
    return {
      moveTo: false,
      pushBoulder: false,
      mapString: '',
      curLevel: 0,
      curSubLevel: 0,
      doBothSubLevels: false,
      flipHorizontally: false,
      fh: false, // Tracking flipping horizontally as done by random separately
      flipVertically: false,
      fv: false, // Tracking flipping vertically as done by random separately
      flipRandomly: false,
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
      if (this.flipRandomly) {
        this.fh = Math.random() < 0.5;
        this.fv = Math.random() < 0.5;
      }
      if (this.fh) {
        this.map = this.map.map((row) => row.split('').reverse().join(''));
      }
      if (this.fv) {
        this.map = this.map.reverse()
      }
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
    getMoveTo(evt) {
      return (evt.getModifierState('Control') || evt.getModifierState('Shift') || this.moveTo);
    },
    getPushTo(evt) {
      return evt.getModifierState('Shift');
    },
    keyboardEvent (evt) {
      console.log('evt', evt);
      if (evt.key === 'm' || evt.key === 'g') this.moveTo = true;
      if (evt.key === 'j' || evt.key === '2' || evt.key === 'ArrowDown') this.move(0, 1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'k' || evt.key === '8' || evt.key === 'ArrowUp') this.move(0, -1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'h' || evt.key === '4' || evt.key === 'ArrowLeft') this.move(-1, 0, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'l' || evt.key === '6' || evt.key === 'ArrowRight') this.move(1, 0, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'y' || evt.key === '7') this.move(-1, -1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'u' || evt.key === '9') this.move(1, -1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'b' || evt.key === '1') this.move(-1, 1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'n' || evt.key === '3') this.move(1, 1, this.getMoveTo(evt), this.getPushTo(evt));
      if (evt.key === 'r') this.loadMap();
    },
    move (x, y, moveTo, pushBoulder) {
      this.pushBoulder = pushBoulder;
      this.moveTo = moveTo;
      if (this.canMoveTo(this.playerPosition[0] + x, this.playerPosition[1] + y, x, y)) {
        this.playerPosition[0] += x;
        this.playerPosition[1] += y;
        this.updateMapString();
        if (moveTo) {
          // repeat same move while possible
          this.move(x,y, this.moveTo, this.pushBoulder);
        }
      } else {
        this.moveTo = false;
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
      } else if (movingTo === '0' && (this.pushBoulder || !this.moveTo)) {
        return this.tryPushBoulder(x, y, dx, dy)
      }
      return false
    },
    getFlippedBaseMap() {
      let m = maps[this.curLevel][this.curSubLevel].slice()
      if (this.fh || this.flipHorizontally) {
        m = m.map((row) => row.split('').reverse().join(''));
      }
      if (this.fv || this.flipVertically) {
        m = m.reverse();
      }
      return m;
    },
    tryPushBoulder (x, y, dx, dy) {
      // Only push boulders once with shift movement
      this.pushBoulder = false;
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
      const prevPosition = this.charAt(this.getFlippedBaseMap(), x, y);
      if (prevPosition === '0' || prevPosition === '.' || prevPosition === '^') {
        this.map[y] = this.map[y].substr(0, x) + '.' + this.map[y].substr(x + 1);
      } else {
        // This prevents stairs from being deleted when a boulder is moved on them
        this.map[y] = this.map[y].substr(0, x) + this.charAt(this.getFlippedBaseMap(), x, y) + this.map[y].substr(x + 1);
      }
      return true;
    }
  },
  watch: {
    flipVertically: function (val) {
      if (val) {
        this.flipRandomly = false;
      }
      this.fv = val;
      this.loadMap();
    },
    flipHorizontally: function (val) {
      if (val) {
        this.flipRandomly = false;
      }
      this.fh = val;
      this.loadMap();
    },
    flipRandomly: function (val) {
      if (val) {
        this.flipVertically = false;
        this.flipHorizontally = false;
      }
      this.loadMap();
    },
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
