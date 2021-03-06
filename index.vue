<template lang="html">
  <v-expansion-panel>
    <v-expansion-panel-content v-for="(item, i) in classes" :key="i">
      <template v-slot:header>
        <div>{{ item.name }}</div>
      </template>
      <v-card>
        <v-dialog v-model="compareData.dialog" max-width="75%">
          <v-card>
            <v-card-text>
              <v-data-table
                class="elevation-1"
                :headers="compareHeaders"
                :hide-actions="true"
                :items="compareData.items"
                :no-data-text="$t('$.algorithms.compare_form.noResultText')"
              >
                <template v-slot:items="props">
                  <td>{{ props.item.id }}</td>
                  <td>{{ props.item.name }}</td>
                  <td>{{ props.item.description }}</td>
                  <td>{{ props.item.ticks }}</td>
                  <td>{{ props.item.processors }}</td>
                </template>
              </v-data-table>
            </v-card-text>
          </v-card>
        </v-dialog>
        <v-dialog v-model="newAlgorithmData.dialog" max-width="50%" persistent>
          <v-card>
            <v-card-title>
              <span class="headline">{{
                $t("$.algorithms.newAlgorithm_form.header")
              }}</span>
            </v-card-title>
            <v-card-text>
              <v-form ref="newAlgorithmForm">
                <v-text-field
                  v-model="newAlgorithmData.name"
                  :label="$t('$.algorithms.newAlgorithm_form.name')"
                  :readonly="newAlgorithmData.loading"
                  :rules="rules.name"
                />
                <v-text-field
                  v-model="newAlgorithmData.description"
                  :label="$t('$.algorithms.newAlgorithm_form.description')"
                  :readonly="newAlgorithmData.loading"
                  :rules="rules.description"
                />
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-spacer />
              <v-btn
                :disabled="newAlgorithmData.loading"
                flat
                @click="newAlgorithmData.dialog = false"
              >
                {{ $t("$.algorithms.newAlgorithm_form.cancel") }}
              </v-btn>
              <v-btn
                color="primary"
                :loading="newAlgorithmData.loading"
                @click="addNewAlgorithm()"
              >
                {{ $t("$.algorithms.newAlgorithm_form.submit") }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="editAlgorithmData.dialog" max-width="50%" persistent>
          <v-card>
            <v-card-title>
              <span class="headline">
                {{
                  $t("$.algorithms.editAlgorithm_form.header", {
                    n: editAlgorithmData.origin
                      ? editAlgorithmData.origin.id
                      : "?",
                  })
                }}
              </span>
            </v-card-title>
            <v-card-text>
              <v-form ref="editAlgorithmForm">
                <v-text-field
                  v-model="editAlgorithmData.name"
                  :label="$t('$.algorithms.editAlgorithm_form.name')"
                  :readonly="editAlgorithmData.loading"
                  :rules="rules.name"
                />
                <v-text-field
                  v-model="editAlgorithmData.description"
                  :label="$t('$.algorithms.editAlgorithm_form.description')"
                  :readonly="editAlgorithmData.loading"
                  :rules="rules.description"
                />
              </v-form>
            </v-card-text>
            <v-card-actions>
              <v-spacer />
              <v-btn
                :disabled="editAlgorithmData.loading"
                flat
                @click="editAlgorithmData.dialog = false"
              >
                {{ $t("$.algorithms.editAlgorithm_form.cancel") }}
              </v-btn>
              <v-btn
                color="primary"
                :loading="editAlgorithmData.loading"
                @click="completeEditAlgorithm()"
              >
                {{ $t("$.algorithms.editAlgorithm_form.submit") }}
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-data-table
          class="elevation-1"
          :headers="algorithmHeaders"
          :hide-actions="true"
          :items="sorted(item.name)"
          :no-data-text="$t('$.algorithms.noResultText')"
        >
          <template v-slot:headers="props">
            <tr>
              <th>
                <v-btn
                  :disabled="algorithmData.selected.length !== 2"
                  flat
                  :loading="compareData.loading"
                  small
                  @click="compareAlgorithms()"
                >
                  {{ $t("$.algorithms.compare") }}
                </v-btn>
              </th>
              <th
                v-for="header in props.headers.filter((h) => h.text)"
                :key="header.text"
                class="text-xs-left"
              >
                {{ header.text }}
              </th>
              <th>
                <v-btn
                  :disabled="!stateUser"
                  flat
                  small
                  @click="newAlgorithmData.dialog = true"
                >
                  {{ $t("$.algorithms.add") }}
                </v-btn>
              </th>
            </tr>
          </template>
          <template v-slot:items="props">
            <td>
              <v-layout align-center fill-height justify-center row>
                <v-switch
                  :disabled="compareData.loading"
                  hide-details
                  :loading="
                    compareData.loading &&
                      algorithmData.selected.includes(props.item)
                  "
                  :value="algorithmData.selected.includes(props.item)"
                  @change="selectAlgorithm(props.item)"
                />
              </v-layout>
            </td>
            <td>{{ props.item.id }}</td>
            <td>{{ props.item.name }}</td>
            <td>{{ props.item.description }}</td>
            <td>{{ props.item.determinantsCount }}</td>
            <td>
              <v-layout align-center fill-height justify-center row>
                <v-icon
                  color="error"
                  :disabled="!stateUser"
                  small
                  @click="deleteAlgorithm(props.item)"
                >
                  delete
                </v-icon>
                <v-icon
                  class="ml-2"
                  :disabled="!stateUser"
                  small
                  @click="beginEditAlgorithm(props.item)"
                >
                  edit
                </v-icon>
                <v-icon
                  class="ml-2"
                  small
                  @click="openApproximaionPage(props.item)"
                >
                  functions
                </v-icon>
                <nuxt-link
                  class="ml-2"
                  :to="`/algorithms/${props.item.id}/determinants`"
                >
                  <v-icon small>
                    remove_red_eye
                  </v-icon>
                </nuxt-link>
              </v-layout>
            </td>
          </template>
        </v-data-table>
      </v-card>
    </v-expansion-panel-content>
  </v-expansion-panel>
</template>

<script lang="ts">
import axios from "axios";
import {Component, State, Vue, Watch} from "nuxt-property-decorator";

@Component({
  async asyncData() {
    const resId = await axios.get("/algorithms", {
      baseURL: process.env.API_ENDPOINT,
    });
    const result = resId.data.data;
    for (let i = 0; i < result.length; i++) {
      if (["1", "3", "10"].includes(result[i].id)) {
        result[i].groupName = "Решение СЛАУ";
      } else if (["12", "13"].includes(result[i].id)) {
        result[i].groupName = "Умножение матриц";
      } else if (["4", "5"].includes(result[i].id)) {
        result[i].groupName = "Скалярное произведение векторов";
      } else if (["11"].includes(result[i].id)) {
        result[i].groupName = "Вычисление максимального элемента в массиве";
      } else if (["2", "6"].includes(result[i].id)) {
        result[i].groupName = "Решение квадратичных уравнений";
      }
    }
    return {
      algorithmItems: result,
    };
  },
})
export default class AboutPage extends Vue {
  public sorted(name) {
    const result = new Array(0);
    for (let i = 0; i < this.algorithmItems.length; i++) {
      // eslint-disable-next-line eqeqeq
      if (this.algorithmItems[i].groupName == name) {
        result.push(this.algorithmItems[i]);
      }
    }
    return result;
  }

  public get classes() {
    return [
      {
        name: "Решение СЛАУ",
      },
      {
        name: "Умножение матриц",
      },
      {
        name: "Скалярное произведение векторов",
      },
      {
        name: "Вычисление максимального элемента в массиве",
      },
      {
        name: "Решение квадратичных уравнений",
      },
    ];
  }
  public algorithmItems!: any[];

  public algorithmData = {
    selected: [] as any[],
  };

  public newAlgorithmData = {
    description: "",
    dialog: false,
    loading: false,
    name: "",
  };

  public editAlgorithmData = {
    description: "",
    dialog: false,
    loading: false,
    name: "",
    origin: null as any,
  };

  public compareData = {
    dialog: false,
    items: [] as any[],
    loading: false,
  };

  public get rules() {
    return {
      description: [
        (v: string) => !!v || this.$t("$.rules.algorithmDescription.required"),
      ],
      name: [(v: string) => !!v || this.$t("$.rules.algorithmName.required")],
    };
  }

  public get algorithmHeaders() {
    return [
      {},
      {
        text: "ID",
      },
      {
        text: this.$t("$.algorithms.headers.name"),
      },
      {
        text: this.$t("$.algorithms.headers.description"),
      },
      {
        text: this.$t("$.algorithms.headers.determinantsCount"),
      },
      {},
    ];
  }

  public get compareHeaders() {
    return [
      {
        sortable: false,
        text: "ID",
      },
      {
        sortable: false,
        text: this.$t("$.algorithms.headers.name"),
      },
      {
        sortable: false,
        text: this.$t("$.algorithms.headers.description"),
      },
      {
        sortable: false,
        text: this.$t("$.algorithms.headers.ticksCompare"),
      },
      {
        sortable: false,
        text: this.$t("$.algorithms.headers.processorsCompare"),
      },
    ];
  }

  @State("user")
  public stateUser!: string;

  @Watch("newAlgorithmData.dialog")
  public onNewAlgorithmDialogChanged(): void {
    if (!this.newAlgorithmData.dialog) {
      (this.$refs.newAlgorithmForm as any).reset();
    }
  }

  @Watch("editAlgorithmData.dialog")
  public onEditAlgorithmDialogChanged(): void {
    if (!this.editAlgorithmData.dialog) {
      (this.$refs.editAlgorithmForm as any).reset();
    }
  }

  public async compareAlgorithms(): Promise<void> {
    this.compareData.loading = true;
    try {
      const res = await axios.get(
        `/algorithms/${this.algorithmData.selected[1].id}/compare/${
          this.algorithmData.selected[0].id
        }`,
        {
          baseURL: process.env.API_ENDPOINT,
        },
      );
      const {processors, ticks} = res.data;

      this.compareData.items =
        processors === null || ticks === null
          ? []
          : [
              {
                description: this.algorithmData.selected[1].description,
                id: this.algorithmData.selected[1].id,
                name: this.algorithmData.selected[1].name,
                processors:
                  processors > 0
                    ? this.$t("$.algorithms.compare_form.processors>")
                    : processors < 0
                    ? this.$t("$.algorithms.compare_form.processors<")
                    : this.$t("$.algorithms.compare_form.processors="),
                ticks:
                  ticks > 0
                    ? this.$t("$.algorithms.compare_form.ticks<")
                    : ticks < 0
                    ? this.$t("$.algorithms.compare_form.ticks>")
                    : this.$t("$.algorithms.compare_form.ticks="),
              },
              {
                description: this.algorithmData.selected[0].description,
                id: this.algorithmData.selected[0].id,
                name: this.algorithmData.selected[0].name,
                processors:
                  processors > 0
                    ? this.$t("$.algorithms.compare_form.processors<")
                    : processors < 0
                    ? this.$t("$.algorithms.compare_form.processors>")
                    : this.$t("$.algorithms.compare_form.processors="),
                ticks:
                  ticks > 0
                    ? this.$t("$.algorithms.compare_form.ticks>")
                    : ticks < 0
                    ? this.$t("$.algorithms.compare_form.ticks<")
                    : this.$t("$.algorithms.compare_form.ticks="),
              },
            ];

      this.compareData.dialog = true;
    } catch (error) {
      throw error;
    } finally {
      this.compareData.loading = false;
    }
  }

  public selectAlgorithm(algorithm: any): void {
    const index = this.algorithmData.selected.indexOf(algorithm);
    if (index >= 0) {
      this.algorithmData.selected.splice(index, 1);
    } else {
      this.algorithmData.selected.unshift(algorithm);
      if (this.algorithmData.selected.length > 2) {
        this.algorithmData.selected.pop();
      }
    }
  }

  public async addNewAlgorithm(): Promise<void> {
    if (!(this.$refs.newAlgorithmForm as any).validate()) {
      return;
    }
    this.newAlgorithmData.loading = true;
    try {
      const resId = await axios.post(
        "/algorithms",
        {
          description: this.newAlgorithmData.description,
          name: this.newAlgorithmData.name,
        },
        {
          baseURL: process.env.API_ENDPOINT,
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        },
      );
      this.algorithmItems.sort((x, y) => {
        const xId = parseInt(x.id, 10);
        const yId = parseInt(y.id, 10);
        return xId < yId ? -1 : xId > yId ? 1 : 0;
      });

      this.newAlgorithmData.dialog = false;
    } catch (error) {
      throw error;
    } finally {
      this.newAlgorithmData.loading = false;
    }
  }

  public async deleteAlgorithm(algorithm: any) {
    if (confirm(this.$t("$.algorithms.deleteConfirm") as string)) {
      const index = this.algorithmItems.indexOf(algorithm);
      this.algorithmItems.splice(index, 1);

      const indexSelected = this.algorithmData.selected.indexOf(algorithm);
      if (indexSelected >= 0) {
        this.algorithmData.selected.splice(indexSelected, 1);
      }

      await axios.delete(`/algorithms/${algorithm.id}`, {
        baseURL: process.env.API_ENDPOINT,
        headers: {
          Authorization: `Bearer ${localStorage.getItem("token")}`,
        },
      });
    }
  }

  public beginEditAlgorithm(algorithm: any): void {
    this.editAlgorithmData.origin = algorithm;
    this.editAlgorithmData.name = algorithm.name;
    this.editAlgorithmData.description = algorithm.description;
    this.editAlgorithmData.dialog = true;
  }

  public async completeEditAlgorithm(): Promise<void> {
    if (!(this.$refs.editAlgorithmForm as any).validate()) {
      return;
    }

    const data = {
      description:
        this.editAlgorithmData.origin.description ===
        this.editAlgorithmData.description
          ? undefined
          : this.editAlgorithmData.description,
      name:
        this.editAlgorithmData.origin.name === this.editAlgorithmData.name
          ? undefined
          : this.editAlgorithmData.name,
    };

    if (Object.values(data).every((v) => v === undefined)) {
      this.editAlgorithmData.dialog = false;
      return;
    }

    this.editAlgorithmData.loading = true;
    try {
      await axios.patch(
        `/algorithms/${this.editAlgorithmData.origin.id}`,
        data,
        {
          baseURL: process.env.API_ENDPOINT,
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        },
      );

      this.editAlgorithmData.origin.name = this.editAlgorithmData.name;
      this.editAlgorithmData.origin.description = this.editAlgorithmData.description;
      this.editAlgorithmData.dialog = false;
    } catch (error) {
      throw error;
    } finally {
      this.editAlgorithmData.loading = false;
    }
  }

  public openApproximaionPage(algorithm: any): void {
    const degree = parseInt(
      prompt(this.$t("$.algorithms.approximationPrompt") as string, "2") || "",
      10,
    );
    if (!isNaN(degree) && degree >= 0) {
      (this as any).$router.push(
        `/algorithms/${algorithm.id}/determinants/approximation/${degree}`,
      );
    }
  }
}
</script>
