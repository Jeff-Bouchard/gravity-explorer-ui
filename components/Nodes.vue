<template>
  <div class="container">
    <tabs-and-search @query-update="queryUpdate"></tabs-and-search>
    <table-block
      ref="table"
      class="nodes-table"
      style="margin-bottom: 34px;"
      :is-loading="isLoading"
      @load-more="pageUpdate"
    >
      <template v-slot:head>
        <tr>
          <th style="width: 35px;"></th>
          <th style="width: 265px;">
            <span>
              Name (
              <icon
                image="/img/icons/сonsul.svg"
                style="top: 3px; margin-top: -6px;"
              ></icon>
              — Consul )
            </span>
          </th>
          <th class="d-none-lg" style="width: 30%;">About</th>
          <th class="header-nodes">Score (0-10)</th>
          <th class="header-nodes">Deposit chain</th>
          <th class="header-nodes">Node deposit</th>
          <th class="header-nodes">Joined</th>
        </tr>
      </template>
      <template v-slot:body>
        <tr
          v-for="(node) in mappedNodesList"
          :key="node.address"
          @click="handleNodeSelect(node)"
        >
          <th style="width: 35px;">
            <icon
              image="/img/icons/сonsul.svg"
              style="top: 1px;"
            ></icon>
          </th>
          <td style="width: 265px; font-size: 13px;">
            <div class="table-icon-cont">
              <table-avatar-icon>
                <icon image="/img/icons/nodes_table.svg"></icon>
              </table-avatar-icon>
              <span style="margin: auto 0;">
              {{ node.name }}
              </span>
            </div>
          </td>
          <td style="width: 30%;" class="d-none-lg">
            {{ node.description }}
          </td>
          <td class="text-green">
            {{ node.score }}
            <!-- <icon
              v-if="index === 1"
              image="/img/icons/сonnect_node.svg"
              style="top: 5px;"
            ></icon> -->
          </td>
          <td class="text-position">{{ node.deposit_chain }}</td>
          <td class="text-position">{{ node.deposit_amount }}</td>
          <td class="text-position">{{ node.joined_at }}</td>
        </tr>
      </template>
    </table-block>
    <client-only>
      <modal v-if="currentNode" name="modal-content-node">
        <modal-content-node
          :rating="String(currentNode.score)"
          :modal-head="currentNode.name"
          :card-date="currentNode.joined_at"
          :card-deposit="currentNode.deposit_amount"
          :card-avatar="currentNode.avatar"
          :nodes-list="currentNodeNebulas"
          caption="Nebulae List:"
          :data="{
            name: currentNode.name,
            address: currentNode.address,
            description: currentNode.description,
            website: {
              name: 'website',
              href: '#',
            },
          }"
        >
        </modal-content-node>
      </modal>
    </client-only>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import TableBlock from '~/components/Table.vue'
import TabsAndSearch from '~/components/TabsAndSearch.vue'
import Icon from '~/components/Icon.vue'
import TableAvatarIcon from '~/components/TableAvatarIcon.vue'
import { Nebula } from '~/models/model/nebula'
import { Node } from '~/models/model/node'
import { NodeDataProvider, mapNode } from '~/data/providers/node'
import { NebulaDataProvider } from '~/data/providers/nebula'
import { FetchCommand } from '~/data/global'
import { CurrencyFormatter } from '../misc/format'

export default Vue.extend({
  name: 'Nodes',
  components: {
    TableBlock,
    TabsAndSearch,
    Icon,
    TableAvatarIcon,
  },
  // eslint-disable-next-line vue/require-prop-types
  props: ['nodesList', 'isLoading'],
  data() {
    return {
      command: { page: 0 } as FetchCommand,
      currentNode: null as Node | null,
      currentNodeNebulas: [] as Nebula[],
    }
  },
  // eslint-disable-next-line vue/require-prop-types
  computed: {
    mappedNodesList() {
      return (this.nodesList as Node[]).map(mapNode)
    },
  },
  methods: {
    queryUpdate(query: string) {
      // @ts-ignore
      this.$refs.table.$el.querySelector('tbody').scrollTo(0, 0)
      this.command = { query, page: 0 }
      this.$emit('query-update', this.command)
    },
    pageUpdate() {
      this.command.page = Number(this.command.page || 0) + 1
      this.$emit('query-update', this.command)
    },
    handleNodeSelect(node: Node) {
      this.currentNode = node
      this.fetchNodeNebulas(node)

      this.$modal.push('modal-content-node')
    },
    fetchNodeNebulas(node: Node) {
      // @ts-ignore
      if (!node.nebulas_using) {
        this.currentNodeNebulas = []
        return
      }

      Promise.all(
        // @ts-ignore
        node.nebulas_using.map((address) => {
          return NebulaDataProvider.fetchExactNebula(address)
        }) as Nebula[]
      ).then((list) => {
        this.currentNodeNebulas = list.filter(Boolean).map((nebula) => {
          return {
            ...nebula,
            // @ts-ignore
            type: CurrencyFormatter.formatChain(nebula.deposit_chain),
            count: String(nebula.score),
            // @ts-ignore
            amount: CurrencyFormatter.formatSubFee(nebula.deposit_chain, Number(Number(nebula.subscription_fee) / 1e8)),
          }
        })
      })
    },
  },
})
</script>

<style lang="scss">
.nodes-table td {
  cursor: pointer;
}
.header-nodes{
  text-align: center;
}
.table-icon-cont {
  display: flex;
  flex-direction: row;
  
  & > :nth-child(1) {
    top: 0;
    margin: auto;
    margin-left: 0;
    margin-right: 8px;
  }
}
.d-none-lg{
  font-size: 13px;
}
.text-position{
  font-size: 13px;
}
</style>
