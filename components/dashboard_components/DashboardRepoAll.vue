<template>
<v-row justify="center">
      <v-dialog
        v-model="dialog"
        persistent
        max-width="600px"
      >
      <v-card>
          <v-card-title>
            <span class="headline">Choose Your Repo To Sell</span>
          </v-card-title>
          <v-card-text>
          <v-card
    class="mx-auto"
    max-width="344"
    outlined
    :key="index"
v-for="(item, index) in items"
  >
    <v-list-item three-line>
      <v-list-item-content>
        <div class="overline mb-4">
          OVERLINE
        </div>
        <v-list-item-title class="headline mb-1">
          Headline 5
        </v-list-item-title>
        <v-list-item-subtitle>Greyhound divisely hello coldly fonwderfully</v-list-item-subtitle>
      </v-list-item-content>

      <v-list-item-avatar
        tile
        size="80"
        color="grey"
      ></v-list-item-avatar>
    </v-list-item>

    <v-card-actions>
      <v-btn
        outlined
        rounded
        text
      >
        Button
      </v-btn>
    </v-card-actions>
  </v-card>
            <v-row justify="center" class="mt-2">
               <v-btn
      class="mx-2"
      fab
      dark
      large
      color="cyan"
    >
      <v-icon dark>
        fab fa-arrow-left
      </v-icon>
    </v-btn>
                <v-btn
      class="mx-2"
      fab
      dark
      large
      color="cyan"
    >
      <v-icon dark>
        fab fa-arrow-right
      </v-icon>
    </v-btn>
            </v-row>

          </v-card-text>

        </v-card>
      </v-dialog>
    </v-row>
</template>
<script lang="ts">
import { Component, Vue, Prop, Emit } from 'nuxt-property-decorator'
@Component
export default class MyStore extends Vue {
  @Prop({ required: true }) readonly dialog!: void
  @Prop({ required: true }) readonly allRepo!: void

}
</script>


