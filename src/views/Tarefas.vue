<template>
    <Formulario />
    <div class="lista">
        <div class="field">
            <p class="control has-icons-left">
                <input class="input" type="text" placeholder="Digite para filtrar as tarefas" v-model="filtro">
                <span class="icon is-small is-left">
                    <i class="fas fa-search"></i>
                </span>
            </p>
        </div>
        <Tarefa v-for="(tarefa, index) in tarefas" :key="index" :tarefa="tarefa" @aoTarefaClicada="selecionarTarefa" />
        <Box v-if="listaEstaVazia">
            Você não está muito produtivo hoje :(
        </Box>
    </div>
    <Modal :mostrar="tarefaSelecionada != null" v-if="tarefaSelecionada != null">
        <template v-slot:cabecalho>
            <p class="modal-card-title">Editar tarefa</p>
            <button class="delete" @click="cancelarEdicao" aria-label="close"></button>
        </template>
        <template v-slot:corpo>
            <div class="field">
                <label for="descricaoDaTarefa" class="label">Descrição da Tarefa</label>
                <input type="text" class="input" v-model="tarefaSelecionada.descricao" id="descricaoDaTarefa">
            </div>
        </template>
        <template v-slot:rodape>
            <button class="button is-success" @click="alterarTarefa">Salvar alterações</button>
            <button class="button" @click="cancelarEdicao">Cancelar</button>
        </template>
    </Modal>
</template>

<script lang="ts">
import { computed, defineComponent, ref, watch } from 'vue';
import Formulario from '../components/Formulario.vue';
import Tarefa from '../components/Tarefa.vue';
import Box from '../components/Box.vue';
import Modal from '../components/Modal.vue';
import { useStore } from '@/store';
import { ALTERAR_TAREFA, OBTER_PROJETOS, OBTER_TAREFAS } from '@/store/tipo-acoes';
import ITarefa from '@/interfaces/ITarefa';

export default defineComponent({
    name: 'Tarefas',
    components: {
        Formulario,
        Tarefa,
        Box,
        Modal
    },
    data() {
        return {
            tarefaSelecionada: null as ITarefa | null
        }
    },
    computed: {
        listaEstaVazia () : boolean {
            return this.tarefas.length === 0
        }
    },
    methods: {
        selecionarTarefa (tarefa: ITarefa) {
            this.tarefaSelecionada = tarefa
        },
        cancelarEdicao () {
            this.tarefaSelecionada = null
        },
        alterarTarefa () {
            this.store
                .dispatch(ALTERAR_TAREFA, this.tarefaSelecionada)
                .then(() => this.tarefaSelecionada = null)
        }
    },
    setup() {
        const store = useStore()
        store.dispatch(OBTER_PROJETOS)
        store.dispatch(OBTER_TAREFAS)

        const filtro = ref('')

        watch(filtro, (valorAtual: string, valorAntigo: string) => {
            if (valorAtual !== valorAntigo)
                store.dispatch(OBTER_TAREFAS, valorAtual)
        })

        return {
            store,
            filtro,
            tarefas: computed(() => store.state.tarefa.tarefas)
        }
    }
});
</script>