<i18n src="@/data/i18n/GearCateTrans.yaml" />
<i18n>
zh_CN:
  weapon: 武器
  hand: 饰品
  body: 护甲
  head: 头饰
  Level: 等级
  Label: 自定义标签
  Enchanted: 神秘
  GearEdit: 编辑装备
  Done: 完成
</i18n>

<template>
  <div>
    <v-btn
      @click.stop="showDialog = true"
      dark
      large
      :color="value.markColor || 'secondary'"
    >
      {{ value.label || $t(eg.name) }} {{ "Lv." + eg.lvl }}<br />
      [{{ eg.p[0] }} {{ eg.p[1] }} {{ eg.p[2] }} {{ eg.p[3] }}]
      <v-icon v-if="eg.isEnchanted">military_tech</v-icon>
    </v-btn>
    <v-dialog v-model="showDialog" max-width="500">
      <v-card>
        <v-card-title>
          <v-col v-if="editableLabel">
            <v-text-field :label="$t('Label')" v-model="AE.label">
            </v-text-field>
          </v-col>
          <v-col v-else> {{ $t("GearEdit") }}</v-col>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="7">
                <v-select
                  v-model="ig.name"
                  :items="availableGearList"
                  :label="translatedTypeList"
                ></v-select>
              </v-col>
              <v-col cols="2">
                <v-text-field
                  :label="$t('Level')"
                  v-model.number="ig.lvl"
                  type="number"
                ></v-text-field>
              </v-col>
              <v-col cols="3">
                <v-checkbox v-model="ig.isEnchanted" :label="$t('Enchanted')">
                </v-checkbox>
              </v-col>
            </v-row>
            <v-row>
              <v-col v-for="i in 4" :key="'p' + i">
                <v-text-field
                  :label="'P' + i"
                  v-model.number="ig.p[i - 1]"
                  type="number"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-btn text color="primary" @click="onDone()">
            {{ $t("Done") }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts">
import { Component, Prop, Vue, Watch } from "vue-property-decorator";
import { GearCate } from "@/data/GearCate";
import { ArsenalEntry } from "@/mechanism/build/ArsenalEntry";
import { Gear } from "@/mechanism/build/Gear";

@Component({})
export default class EditorArsenalEntry extends Vue {
  showDialog = false;
  @Prop() value!: ArsenalEntry | Gear;
  @Prop({ default: () => ["weapon", "hand", "body", "head"] }) types!: string[];
  @Prop({ default: false }) editableLabel!: boolean;
  @Prop({ default: false }) editableColor!: boolean;
  AE: ArsenalEntry = new ArsenalEntry();

  mounted() {
    if (this.value instanceof Gear) {
      this.AE.gear.load(this.value);
      this.editableColor = false;
      this.editableLabel = false;
    } else this.AE.load(this.value);
  }

  // means internal gear
  get ig() {
    return this.AE.gear;
  }

  // means external gear
  get eg() {
    return this.value instanceof Gear ? this.value : this.value.gear;
  }

  @Watch("value", { deep: true })
  onValueChanged() {
    this.value instanceof Gear
      ? this.ig.load(this.value)
      : this.AE.load(this.value);
  }

  onDone() {
    this.showDialog = false;
    this.$emit("input", this.value instanceof Gear ? this.ig : this.AE);
  }

  get availableGearList() {
    const ret = new Array<{ text: string; value: string }>(0);
    for (const key in GearCate) {
      if (
        this.types.length == 0 ||
        this.types.find(e => e == GearCate[key].type)
      ) {
        ret.push({
          text: this.$t(GearCate[key].name).toString(),
          value: GearCate[key].name
        });
      }
    }
    return ret;
  }

  get translatedTypeList() {
    const ret = new Array<string>(0);
    this.types.forEach(e => ret.push(this.$t(e).toString()));
    return ret.join(" / ");
  }
}
</script>

<style scoped></style>
