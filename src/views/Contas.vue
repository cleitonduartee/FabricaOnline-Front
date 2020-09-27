<template>
  <div>
    <!-- Formulario -->
    <v-card class="my-9 mx-auto" width="50%" v-if="mostrarForm">
      <v-btn absolute dark fab top right color="red" @click="cancelar" small>
        <v-icon>
          mdi-close
        </v-icon>
      </v-btn>
      <v-container>
        <div class="mb-4">Nova Conta</div>
        <v-row>
          <v-col cols="12" sm="6">
            <v-text-field
              label="Descrição"
              outlined
              v-model="contaAtual.descricao"
              hide-details
            >
            </v-text-field>
          </v-col>
          <v-col cols="12" sm="6">
            <v-select
              :items="tipo"
              label="Tipo"
              outlined
              v-model="contaAtual.tipo"
              hide-details
            >
            </v-select>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12" sm="6">
            <v-text-field
              label="Valor"
              outlined
              type="number"
              v-model="contaAtual.valor"
              hide-details
            >
            </v-text-field>
          </v-col>
          <v-col cols="12" sm="6">
            <v-menu
              ref="menudataDialog"
              v-model="menudataDialog"
              :close-on-content-click="false"
              transition="scale-transition"
              offset-y
              max-width="290px"
              min-width="290px"
            >
              <template v-slot:activator="{ on }">
                <v-text-field
                  v-model="dataFormatada"
                  label="Data"
                  persistent-hint
                  prepend-icon="mdi-calendar"                  
                  v-on="on"
                ></v-text-field>
              </template>
              <v-date-picker
                v-model="contaAtual.data"
                no-title
                @input="menu1 = false"
              ></v-date-picker>
            </v-menu>
            <p>
              Date in ISO format: <strong>{{ contaAtual.data }}</strong>
            </p>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="12">
            <v-textarea
              outlined
              label="Observações"
              v-model="contaAtual.observacoes"
            ></v-textarea>
          </v-col>
        </v-row>
        <v-card-actions>
          <v-row justify="center" class="mb-3 mt-0">
            <v-btn color="success" @click="salvar" class="mx-1">Salvar</v-btn>
            <v-btn color="warning" @click="cancelar" class="mx-1"
              >Cancelar</v-btn
            >
          </v-row>
        </v-card-actions>
      </v-container>
    </v-card>
    <!-- FIm Formulario -->

    <!-- lista de contas -->
    <v-data-table
      :headers="cabecalhoContas"
      :items="contas"
      class="elevation-1"
    >
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>Contas Cadastrados</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="abrirFormulario" v-if="!mostrarForm"
            >Novo Conta</v-btn
          >
        </v-toolbar>
      </template>
      <template v-slot:item.tipo="{ item }">
        <div :class="item.tipo == 'Receita' ? 'green--text' : 'red--text'">
          {{ item.tipo }}
        </div>
      </template>
      <template v-slot:item.valor="{ item }">
        R$ {{ item.valor.toFixed(2) }}
      </template>
      <template v-slot:item.acoes="{ item }">
        <v-icon small class="mr-2" @click="detalhes(item)" color="blue"
          >mdi-magnify</v-icon
        >
        <v-icon small @click="excluirConfirm(item)" color="red"
          >mdi-trash-can</v-icon
        >
      </template>
      <template v-slot:no-data>
        <v-row justify="center">
          <v-subheader>Nenhuma conta cadastrada</v-subheader>
        </v-row>
      </template>
      <template v-slot:body.append>
        <v-toolbar flat color="white">
          <v-toolbar-title> Balanço: </v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-card flat :class="balancoGeral >= 0 ? 'green--text' : 'red--text'">
            R$ {{ balancoGeral.toFixed(2) }}
          </v-card>
        </v-toolbar>
      </template>
    </v-data-table>
    <!--FIM lista de contas -->

    <!-- Janela de confirmação de exclusão -->
    <v-dialog v-model="dialogConfirmaExclusao" max-width="455" persistent>
      <v-card>
        <v-card-title class="headline"
          >Deseja Realmente excluir essa conta?</v-card-title
        >

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn color="green darken-1" text @click="cancelar">
            Não
          </v-btn>

          <v-btn color="red darken-1" text @click="confirmaExclusao">
            Sim
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <!-- Janela de Detalhes -->
    <v-dialog v-model="dialogDetalhes" max-width="40%">
      <v-card>
        <v-card-title class="headline">Detalhes da conta</v-card-title>

        <v-card-text>
          <v-row>
            <v-col cols="12" sm="">
              <v-text-field
                label="Descrição"
                outlined
                disabled
                :value="contaAtualDetalhes.descricao"
                hide-details
              >
              </v-text-field>
            </v-col>
            <v-col cols="12" sm="4">
              <v-select
                :items="tipo"
                label="Tipo"
                outlined
                disabled
                :value="contaAtualDetalhes.tipo"
                hide-details
              >
              </v-select>
            </v-col>
            <v-col cols="12" sm="4">
              <v-text-field
                label="Valor"
                outlined
                type="number"
                disabled
                :value="
                  contaAtualDetalhes.valor
                    ? contaAtualDetalhes.valor.toFixed(2)
                    : contaAtualDetalhes.valor
                "
                hide-details
              >
              </v-text-field>
            </v-col>
          </v-row>
          <v-row>
            <v-col cols="12">
              <v-textarea
                outlined
                label="Observações"
                disabled
                :value="contaAtualDetalhes.observacoes"
              ></v-textarea>
            </v-col>
          </v-row>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>

          <v-btn color="green darken-1" text @click="cancelar">
            Fechar
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      dialogConfirmaExclusao: false,
      dialogDetalhes: false,
      contaAExcluir: null,
      menudataDialog: false,
      balancoGeral: 0,
      dataFormatada:'',
      geradorId: 4,
      itemEdicao: false,
      mostrarForm: false,
      contaAtual: {},
      contaAtualDetalhes: {},
      cabecalhoContas: [
        {
          text: "Descrição",
          value: "descricao"
        },
        {
          text: "Tipo",
          value: "tipo"
        },
        {
          text: "Valor",
          value: "valor"
        },
        {
          text: "Data",
          value: "data"
        },
        {
          text: "Ações",
          value: "acoes",
          sortable: false
        }
      ],
      contas: [],
      tipo: ["Receita", "Despesa"]
    };
  },
  created() {
    this.initContas();
  },
  methods: {
    initContas() {
      (this.contas = [
        {
          id: 0,
          descricao: "Conta 1",
          observacoes: "Teste",
          tipo: "Receita",
          valor: 10.0
        },
        {
          id: 1,
          descricao: "Conta 2",
          observacoes: "Testando",
          tipo: "Receita",
          valor: 20.0
        },
        {
          id: 2,
          descricao: "Conta 3",
          observacoes: "teste 2 ",
          tipo: "Despesa",
          valor: 30.0
        },
        {
          id: 3,
          descricao: "Conta 4 ",
          observacoes: "Testando 2",
          tipo: "Receita",
          valor: 40.0
        }
      ]),
        this.calcularBalanco();
    },
    calcularBalanco() {
      this.balancoGeral = 0;
      this.contas.forEach(conta => {
        if (conta.tipo == "Receita") {
          this.balancoGeral += conta.valor;
        } else {
          this.balancoGeral -= conta.valor;
        }
      });
    },
    salvar() {
      this.contaAtual.id = this.geradorId;
      this.geradorId++;
      this.contaAtual.valor = parseFloat(this.contaAtual.valor);
      this.contas.push(Object.assign({}, this.contaAtual));
      this.calcularBalanco();
      this.cancelar();
    },
    cancelar() {
      this.mostrarForm = false;
      this.contaAtual = {};
      this.contaAExcluir = null;
      this.dialogConfirmaExclusao = false;
      this.dialogDetalhes = false;
    },
    abrirFormulario() {
      this.mostrarForm = true;
    },

    excluirConfirm(conta) {
      this.contaAExcluir = conta;
      this.dialogConfirmaExclusao = true;
    },
    confirmaExclusao() {
      this.contas.forEach((conta, index) => {
        if (conta.id == this.contaAExcluir.id) {
          this.contas.splice(index, 1);
        }
      });
      this.cancelar();
      this.calcularBalanco();
    },
    detalhes(conta) {
      // alert(JSON.stringify(conta))
      this.dialogDetalhes = true;
      this.contaAtualDetalhes = conta;
    }
  }
};
</script>

<style>
</style>