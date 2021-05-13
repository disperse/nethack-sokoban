<template>
  <div id="app" :class="{'invert':invert}">
    <h1 v-if="!won">Sokoban Level {{ (this.curLevel + 1) + ((this.curSubLevel === 0) ? 'a' : 'b')}}</h1>
    <Map v-if="!won" :map-string="mapString" :player-position="playerPosition" :invert="invert"/>
    <h3>Push R to restart.</h3>
    <h1 v-if="won">YOU WIN!</h1>
    <div>
      <div v-if="trackTurnsLocally" class="inline">
        <h3>Scores</h3>
        <div>
          <ul>
            <li>Turns: {{ curTurns }}</li>
            <li>Best turns: {{ (bestTurns[curLevel][curSubLevel] === Number.MAX_SAFE_INTEGER) ? 'None' : bestTurns[curLevel][curSubLevel] }}</li>
            <li>Time: {{ timeElapsed }}s</li>
            <li>Best time: {{ (bestTime[curLevel][curSubLevel] === Number.MAX_SAFE_INTEGER) ? 'None' : bestTime[curLevel][curSubLevel] + 's' }}</li>
          </ul>
        </div>
      </div>
      <div class="inline">
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
        <div>
          <input type="checkbox" v-model="invert" id="invert"><label for="invert">Invert colors</label>
        </div>
        <div>
          <input type="checkbox" v-model="trackTurnsLocally" id="trackTurnsLocally"><label for="trackTurnsLocally">Track score locally</label>
        </div>
      </div>
      <div class="inline">
        <h3>Levels</h3>
        <ul>
          <li v-for="level in levelListing" v-bind:key="level.name"><a href="#" @click="changeLevel(level)">{{level.name}}</a></li>
        </ul>
      </div>
    </div>
    <div>
      <h3>More info</h3>
      <p>Source code available at <a href="https://github.com/disperse/nethack-sokoban">https://github.com/disperse/nethack-sokoban</a>. Report issues or request new features there.</p>
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
    this.loadOptions();
    this.curSubLevel = (this.doBothSubLevels) ? 0 : Math.floor(Math.random() * maps[this.curLevel].length)
    this.loadMap();
  },
  data () {
    return {
      curTurns: 0,
      bestTurns: [[Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER]],
      bestTime: [[Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER], [Number.MAX_SAFE_INTEGER, Number.MAX_SAFE_INTEGER]],
      timerInterval: null,
      timeStarted: 0,
      timeElapsed: 0,
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
      won: false,
      invert: false,
      trackTurnsLocally: false
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
    loadOptions () {
      if (localStorage.getItem('doBothSubLevels') !== null) {
        this.doBothSubLevels = (localStorage.getItem('doBothSubLevels') === 'true');
      }
      if (localStorage.getItem('flipHorizontally') !== null) {
        this.flipHorizontally = (localStorage.getItem('flipHorizontally') === 'true');
      }
      if (localStorage.getItem('flipVertically') !== null) {
        this.flipVertically = (localStorage.getItem('flipVertically') === 'true');
      }
      if (localStorage.getItem('flipRandomly') !== null) {
        this.flipRandomly = (localStorage.getItem('flipRandomly') === 'true');
      }
      if (localStorage.getItem('invert') !== null) {
        this.invert = (localStorage.getItem('invert') === 'true');
      }
      if (localStorage.getItem('trackTurnsLocally') !== null) {
        this.trackTurnsLocally = (localStorage.getItem('trackTurnsLocally') === 'true');
      }
      if (localStorage.getItem('bestTurns') !== null) {
        this.bestTurns = JSON.parse(localStorage.getItem('bestTurns'));
      }
      if (localStorage.getItem('bestTime') !== null) {
        this.bestTime = JSON.parse(localStorage.getItem('bestTime'));
      }
    },
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
      // reset timer
      clearInterval(this.timerInterval);
      this.timerInterval = setInterval(() => this.timeElapsed = Math.round((Date.now() - this.timeStarted) / 1000), 1000);
      this.curTurns = 0;
      this.timeStarted = Date.now();
      // reset win state
      this.won = false;
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
      // console.log('evt', evt);
      // Prevent window from scrolling when using the arrow keys
      if (evt.key === 'ArrowDown' || evt.key === 'ArrowUp' || evt.key === 'ArrowLeft' ||evt.key === 'ArrowRight') evt.preventDefault();
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
        this.curTurns++;
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
          this.bestTurns[this.curLevel][this.curSubLevel] = Math.min(this.bestTurns[this.curLevel][this.curSubLevel], this.curTurns);
          localStorage.setItem('bestTurns', JSON.stringify(this.bestTurns));

          this.bestTime[this.curLevel][this.curSubLevel] = Math.min(this.bestTime[this.curLevel][this.curSubLevel], this.timeElapsed);
          localStorage.setItem('bestTime', JSON.stringify(this.bestTime));

          if (this.curLevel === maps.length - 1 && (!this.doBothSubLevels || this.curSubLevel === maps[this.curLevel].length - 1)) {
            this.won = true;
            this.curLevel = 1;
          } else {
            if (this.doBothSubLevels) {
              if (this.curSubLevel === maps[this.curLevel].length - 1) {
                this.curLevel++;
                this.curSubLevel = 0;
              } else {
                this.curSubLevel++;
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
      localStorage.setItem('flipVertically', val);
      if (val) {
        this.flipRandomly = false;
      }
      this.fv = val;
      this.loadMap();
    },
    flipHorizontally: function (val) {
      localStorage.setItem('flipHorizontally', val);
      if (val) {
        this.flipRandomly = false;
      }
      this.fh = val;
      this.loadMap();
    },
    flipRandomly: function (val) {
      localStorage.setItem('flipRandomly', val);
      if (val) {
        this.flipVertically = false;
        this.flipHorizontally = false;
      }
      this.loadMap();
    },
    doBothSubLevels: function (val) {
      localStorage.setItem('doBothSubLevels', val);
      if (val && this.curSubLevel > 0) {
        this.curSubLevel = 0;
        this.loadMap();
      }
    },
    invert: function (val) {
      localStorage.setItem('invert', val);
    },
    trackTurnsLocally: function (val) {
      localStorage.setItem('trackTurnsLocally', val);
    }
  }
}
</script>

<style>
html, body {
  padding: 0;
  margin: 0;
  overflow: hidden;
}

#app {
  margin: 0;
  padding: 2em;
  font-family: 'Menlo', monospace;
  color: white;
  background-color: black;
  height: 100vh;
  overflow: scroll;
  box-sizing: border-box;
}

#app.invert {
  color: black;
  background-color: white
}

a {
  color: white;
}

.inline {
  display: inline-block;
  height: 100%;
  vertical-align: top;
  margin-right: 1.5em;
}

#app.invert a {
  color: black;
}
</style>
