<template>
  <div>
    <v-card  width="50%" class="my-9 mx-auto" v-if="mostrarForm">
      <v-btn absolute dark fab top right color="red" @click="cancelar">
        <v-icon>
            mdi-close
        </v-icon>
      </v-btn>
      <v-container>
        <div class="mb-4">{{formTitulo}}</div>
        <v-row>
          <v-col cols="12" sm="6"  >
            <v-text-field
            label="Nome"           
            outlined
            v-model="usuarioAtual.nome">
            </v-text-field>
          </v-col>
          <v-col cols="12" sm="6"  >
            <v-text-field
            label="E-mail"           
            outlined
            v-model="usuarioAtual.email">
            </v-text-field>
          </v-col>
           <v-col cols="12" sm="6"  >
            <v-text-field
            label="Username"           
            outlined
            v-model="usuarioAtual.username">
            </v-text-field>
          </v-col>
           <v-col cols="12" sm="6"  >
            <v-text-field
            label="Senha"           
            outlined
            type="password"
            v-model="usuarioAtual.senha">
            </v-text-field>
          </v-col>           
        </v-row>
      </v-container>
      <v-card-actions>
        <v-row justify="center">
          <v-btn color="success" @click="atualizar" class="mx-1 " v-if="itemEdicao != null">Atualizar</v-btn>
          <v-btn color="success" @click="salvar" class="mx-1" v-else>Salvar</v-btn>
          <v-btn color="warning" @click="cancelar" class="mx-1">Cancelar</v-btn>
        </v-row>
      </v-card-actions>
    </v-card>
    <v-data-table :headers=" cabecalho" :items="usuarios" class="elevation-1">
      <template v-slot:top>
        <v-toolbar flat color="white">
          <v-toolbar-title>Usuários Cadastrados</v-toolbar-title>
          <v-divider class="mx-4" inset vertical></v-divider>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="abrirFormulario" v-if="!mostrarForm">Novo Usuário</v-btn>
        </v-toolbar>
      </template>
      <template v-slot:item.acoes="{ item }">
        <v-icon small class="mr-2" @click="editItem(item)" color="blue">mdi-pencil</v-icon>
        <v-icon
          small
          @click="ativaInativa(item)"
          :color="item.ativo? 'green':'red'"
        >{{ item.ativo ? " mdi-check-bold": " mdi-cancel"}}</v-icon>
      </template>
      <template v-slot:no-data>
        <v-subheader>Nenhum usuário cadastrado</v-subheader>
      </template>
    </v-data-table>
  </div>
</template>

<script>
export default {
  data: () => ({
    mostrarForm:false,
    itemEdicao: null,
    usuarios: [],
    usuarioAtual: {},
    geradorDeID: 3,
    cabecalho: [
      {
        text: "ID",
        value: "id"
      },
      {
        text: "Nome",
        value: "nome"
      },
      {
        text: "Username",
        value: "username"
      },
      {
        text: "E-mail",
        value: "email"
      },
      {
        text: "Ações",
        value: "acoes"
      }
    ]
  }),

  computed: {
    formTitulo() {
      return this.itemEdicao === null ? "Novo Usuário" : "Editar Usuário";
    }
  },

  created() {
    this.initialize();
  },

  methods: {
    initialize() {
      this.usuarios = [
        {
          id: 0,
          nome: "João da Silva",
          username: "jao01",
          senha: "123",
          email: "jao01@email.com",
          ativo: true
        },
        {
          id: 1,
          nome: "Maria da Silva",
          username: "maria01",
          senha: "123",
          email: "maria01@email.com",
          ativo: false
        },
        {
          id: 2,
          nome: "Zé da Silva",
          username: "ze01",
          senha: "123",
          email: "ze01@email.com",
          ativo: true
        }
      ];
    },
    abrirFormulario() {
      this.mostrarForm = true
    },
    editItem(usuario) {          
     this.mostrarForm = true
     this.itemEdicao = true 
     this.usuarioAtual = Object.assign({},usuario) 
    },
    atualizar(){
      console.log(this.usuarioAtual)
      let achou = false
      let i = 0
      while(i<this.usuarios.length && achou == false){
        if(this.usuarioAtual.id == this.usuarios[i].id){
         this.usuarios.splice(i,1, this.usuarioAtual)
          achou = true
        }
        i++
      }
      this.cancelar()
    },
    deleteItem(usuario) {
      console.log(usuario);
    },
    ativaInativa(usuario) {
      usuario.ativo = !usuario.ativo;
    },
    salvar(){
     this.usuarioAtual.ativo = true,
     this.usuarioAtual.id = this.geradorDeID
     this.usuarios.push(Object.assign({}, this.usuarioAtual)),
     this.geradorDeID++
     this.cancelar()
    },
    cancelar(){
      this.usuarioAtual = {},
      this.itemEdicao = null,
      this.mostrarForm = false
    }
  }
};
</script>