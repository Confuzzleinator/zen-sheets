<template>
  <v-container>
    <h1>Zen Sheets</h1>
    <p>Yell at Confuzzleinator on Discord if anything stops working, or there are additional features you would like.</p>
    <v-card>
      <v-container>
        <v-row>
          <v-col>
            <v-file-input v-model='files' label='Upload file'></v-file-input>
          </v-col>
          <v-col>
            <v-text-field v-model='year' label='Game year (optional)'></v-text-field>
          </v-col>
        </v-row>
        <v-row justify='center'>
          <v-btn v-if='!ready' color='primary' @click='parse' :loading='parsing' :disabled='files===null'>Parse</v-btn>
          <v-btn v-if='ready' color='primary' @click='loadPlayers' :disabled='genDisabled'>Generate Table</v-btn>
        </v-row>
        <v-row v-if='ready' justify='center'>
          <v-data-table disable-pagination hide-default-footer :headers='headers' :items='players'></v-data-table>
        </v-row>
      </v-container>
    </v-card>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      files: null,
      result: "r",
      //reader: new FileReader(),
      parsing: false,
      ready: false,
      genDisabled: false,
      year: null,
      headers: [
        { text: 'Player', value: 'player' },
        { text: 'Position', value: 'pos' },
        { text: 'Year', value: 'year' },
        { text: 'Overall', value: 'ovr' },
        { text: 'Potential', value: 'pot' }
      ],
      players: [
      ]
    }
  },
  methods: {
    async parse() {
      this.parsing = true;
      this.result = await this.files.text();
      this.result = JSON.parse(this.result);
      this.parsing = false;
      this.ready = true;
    },
    loadPlayers() {
      for(let i = 0; i < this.result.players.length; i++) {
        let p = this.result.players[i];
        // Only free agents (tid of -1)
        if(p.tid == -1) {
          this.players.push({
            player: p.userID,
            pos: p.ratings[0].pos,
            // Year or age?
            year: (this.year == null) ? p.born.year : parseInt(this.year) - p.born.year,
            ovr: p.ratings[0].ovr,
            pot: p.ratings[0].pot
          });
        }

      }
      this.genDisabled = true;
    }
  }
}
</script>
