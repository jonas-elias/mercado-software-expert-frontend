<script setup>
</script>

<template>
    <div class="container mx-auto">
        <div class="navbar bg-base-100">
            <div class="navbar-start">
                <div class="dropdown">
                    <label tabindex="0" class="btn btn-ghost lg:hidden">
                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24"
                            stroke="currentColor">
                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                d="M4 6h16M4 12h8m-8 6h16" />
                        </svg>
                    </label>
                </div>
                <img class="btn btn-ghost normal-case text-xl"
                    src="https://www.softexpert.com/wp-content/themes/Zephyr-child/icon-softexpert-site.png"
                    alt="SoftExpert Logo">
            </div>
            <Navbar />
            <div class="navbar-end">
                <a class="btn">GitHub</a>
            </div>
        </div>

        <input type="checkbox" id="modal_insert_venda" class="modal-toggle" />
        <div class="modal">
            <div class="modal-box">
                <h3 class="font-bold text-lg">Adicione uma nova venda</h3>

                <div v-for="(item, index) in produtosGrupo" :key="index">
                    <div class="form-control w-full">
                        <label class="label">
                            <span class="label-text">Produto:</span>
                        </label>
                        <select @click="getValorProduto(item)" v-model="item.id_produto"
                            class="select select-bordered w-full">
                            <option value="0">Selecione um produto</option>
                            <option v-for="tipo in produtos" :key="tipo.id" :value="tipo.id">{{ tipo.nome }}</option>
                        </select>
                    </div>
                    <div class="form-control w-full">
                        <label class="label">
                            <span class="label-text">Quantidade:</span>
                        </label>
                        <input @keyup="getValorTotalProduto(item)" v-model="item.quantidade" type="text"
                            placeholder="Quantidade" class="input input-bordered w-full" />
                    </div>
                    <div class="flex justify-between mt-2">
                        <div>
                            <button @click="addProduct" class="mr-3 text-md">Adicionar</button>
                            <button @click="removeProduct(index)">Remover</button>
                        </div>
                        <span class="text-md">Total produto (com impostos): {{ item.total }}</span>
                    </div>
                </div>

                <div class="mt-5">
                    <span class="text-md">Total venda (sem impostos): R${{ this.total_venda }}</span>
                </div>
                <div>
                    <span class="text-md">Total impostos: R${{ this.total_impostos }}</span>
                </div>
                <div class="modal-action">
                    <label for="modal_insert_venda" class="btn">Fechar</label>
                    <label class="btn" @click="saveVenda()">Salvar</label>
                </div>
            </div>
        </div>

        <div class="flex justify-end">
            <label class="btn btn-outline btn-sm" for="modal_insert_venda">Adicione uma nova venda</label>
        </div>
        <div class="overflow-x-auto">
            <table class="table table-zebra">
                <!-- head -->
                <thead>
                    <tr>
                        <th>Id</th>
                        <th>Total da Compra</th>
                        <th>Total Impostos</th>
                    </tr>
                </thead>
                <tbody>
                    <tr>
                        <th>1</th>
                        <td>Cy Ganderton</td>
                        <td>Quality Control Specialist</td>
                    </tr>
                    <tr>
                        <th>2</th>
                        <td>Hart Hagerty</td>
                        <td>Desktop Support Technician</td>
                    </tr>
                </tbody>
            </table>
        </div>
    </div>
</template>

<script lang="ts">
import Navbar from './Navbar.vue';
import axios from 'axios'

export default {
    components: {
        Navbar,
    },
    data() {
        return {
            produtosGrupo: [
                {
                    produto: "", quantidade: 0, total: 0,
                    valor_item: 0, valor_total: 0, valor_imposto: 0, id_produto: 0
                }
            ],
            apiProdutos: 'http://localhost:8000/api/produto',
            produtos: [],
            total_venda: 0,
            total_impostos: 0,
        };
    },
    mounted() {
        this.getProdutos();
    },
    methods: {
        getProdutos() {
            axios.get(this.apiProdutos).then((res) => {
                this.produtos = res.data['dados']
            }).catch((error) => {
                console.error("Erro na solicitação:", error);
            });
        },
        addProduct() {
            this.produtosGrupo.push({ produto: "", quantidade: 0, total: 0, valor_item: 0, valor_total: 0, valor_imposto: 0, id_produto: 0 });
        },
        removeProduct(index: number) {
            this.produtosGrupo.splice(index, 1);
        },
        saveVenda() {
            console.log(this.produtosGrupo)
        },
        getValorProduto(objeto) {
            if (objeto.id_produto) {
                axios.get(this.apiProdutos + '/' + objeto.id_produto).then((res) => {
                    objeto.valor_total = res.data['dados'][0]['total']
                    objeto.valor_imposto = res.data['dados'][0]['valor_imposto']
                    objeto.valor_item = res.data['dados'][0]['preco']
                    this.getValorProduto(objeto)
                }).catch((error) => {
                    // this.openModalError()
                });
            }
        },
        getValorTotalProduto(objeto) {
            objeto.total = objeto.quantidade * objeto.valor_total

            this.calculaTotalVenda()
        },
        calculaTotalVenda() {
            let total_impostos = 0
            let total_venda = 0
            this.produtosGrupo.forEach(element => {
                total_impostos += element.valor_imposto * element.quantidade
                total_venda += element.valor_item * element.quantidade
                console.log(element)
            });

            this.total_impostos = total_impostos
            this.total_venda = total_venda
        }
    }
};
</script>