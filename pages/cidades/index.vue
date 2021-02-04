<template>
  <div>
    <v-data-table
      :headers="headers"
      :items="cidades"
      class="elevation-1"
      hide-default-footer
      :loading="loading"
      loading-text="Buscando informações..."
    >
      <template #top>
        <v-toolbar flat>
          <v-toolbar-title>Cidades</v-toolbar-title>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template #activator="{ on, attrs }">
              <v-btn color="primary" dark class="mb-2" v-bind="attrs" v-on="on">
                Cadastrar
              </v-btn>
            </template>
            <v-card>
              <v-card-title>
                <span class="headline">{{
                  cidade.id ? 'Alterar cidade' : 'Cadastrar cidade'
                }}</span>
              </v-card-title>

              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="6">
                      <v-text-field
                        v-model="cidade.nome"
                        label="Nome"
                      ></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="6">
                      <v-select
                        v-model="cidade.estadoId"
                        :items="estados"
                        item-text="nome"
                        item-value="id"
                        label="Estado"
                        single-line
                      ></v-select>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="red darken-1" text @click="cancel">
                  Cancelar
                </v-btn>
                <v-btn
                  color="blue darken-1"
                  :loading="loading"
                  :disabled="loading"
                  text
                  @click="createOrUpdateCidade()"
                >
                  {{ cidade.id ? 'Alterar' : 'Cadastrar' }}
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
          <v-dialog v-model="dialogDelete" max-width="500px">
            <v-card>
              <v-card-title class="headline text-center"
                >Tem certeza que deseja remover a cidade
                {{ cidade.nome }}?</v-card-title
              >
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="cancel"
                  >Cancelar</v-btn
                >
                <v-btn color="red darken-1" text @click="deleteCidade()"
                  >SIM</v-btn
                >
                <v-spacer></v-spacer>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
      <template #[`item.created_at`]="{ item }">
        {{ formatDate(item.created_at) }}
      </template>
      <template #[`item.updated_at`]="{ item }">
        {{ formatDate(item.updated_at) }}
      </template>
      <template #[`item.actions`]="{ item }">
        <v-icon color="blue" small class="mr-2" @click="openUpdateCidade(item)">
          mdi-pencil
        </v-icon>
        <v-icon color="red" small @click="openDeleteCidade(item)">
          mdi-delete
        </v-icon>
      </template>
    </v-data-table>
    <v-snackbar
      :color="sncakbarColor"
      :timeout="3000"
      :value="snackbar"
      absolute
      right
      shaped
      bottom
    >
      {{ snackbarText }}.
    </v-snackbar>
  </div>
</template>

<script>
export default {
  data() {
    return {
      loading: false,
      cidades: [],
      estados: [],
      snackbar: false,
      sncakbarColor: '',
      snackbarText: '',
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: 'Nome',
          sortable: false,
          value: 'nome',
        },
        { text: 'Estado', value: 'estado.abreviacao' },
        { text: 'Criado', value: 'created_at' },
        { text: 'Atualizado', value: 'updated_at' },
        { text: 'Ações', value: 'actions', sortable: false },
      ],
      cidade: {
        nome: '',
        estadoId: '',
      },
    };
  },
  created() {
    this.fetchCidades();
    this.fetchEstados();
  },
  methods: {
    async fetchCidades() {
      this.loading = true;
      try {
        const data = await this.$axios.$get('cidades');
        this.cidades = data;
      } catch (err) {
        let msg =
          'Houve uma falha ao carregar os registros. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        return this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async fetchEstados() {
      this.loading = true;
      try {
        const data = await this.$axios.$get('estados');
        this.estados = data;
      } catch (err) {
        let msg =
          'Houve uma falha ao carregar os registros. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        return this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async updateCidade() {
      this.loading = true;
      try {
        await this.$axios.$put(`cidades/${this.cidade.id}`, {
          estadoId: this.cidade.estadoId,
          nome: this.cidade.nome,
        });
        this.fetchCidades();
        this.dialog = false;
        this.cidade = {};
        this.showSnack('Cidade atualizada com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao alterar a cidade. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async createCidade() {
      this.loading = true;
      try {
        await this.$axios.$post('cidades', this.cidade);
        this.fetchCidades();
        this.dialog = false;
        this.cidade = {};
        this.showSnack('Cidade cadastrada com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao cadastrar a cidade. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    async deleteCidade() {
      this.loading = true;
      try {
        await this.$axios.$delete(`cidades/${this.cidade.id}`);
        this.fetchCidades();
        this.dialogDelete = false;
        this.cidade = {};
        this.showSnack('Cidade removida com sucesso', 'green');
      } catch (err) {
        let msg =
          'Houve uma falha ao remover a cidade. Por favor, tente novamente';
        if (err.response?.data?.message) {
          msg = err.response.data.message;
        }
        this.showSnack(msg, 'red');
      }
      this.loading = false;
    },
    showSnack(text, color) {
      this.snackbarText = text;
      this.sncakbarColor = color;
      this.snackbar = true;
    },
    createOrUpdateCidade() {
      if (this.cidade.id) {
        this.updateCidade();
      } else {
        this.createCidade();
      }
    },
    openUpdateCidade(cidade) {
      this.cidade = cidade;
      this.dialog = true;
    },
    openDeleteCidade(cidade) {
      this.cidade = cidade;
      this.dialogDelete = true;
    },
    formatDate(date) {
      const data = new Date(date);
      return ` ${data.toLocaleDateString()}  ${data.toLocaleTimeString()}`;
    },
    cancel() {
      this.cidade = {};
      this.dialog = false;
      this.dialogDelete = false;
    },
  },
};
</script>

<style></style>
