<template>
  <div v-if="stats.global" class="mt-4 bg-blue-800 rounded-lg">
    <div class="relative flex justify-center py-4 text-blue-200 rounded-t-lg heading">
      <svg class="w-6 h-6">
        <use xlink:href="#graph" />
      </svg>
      <span class="mx-4 text-lg font-semibold uppercase">STATS</span>
      <svg class="w-6 h-6" style="transform: scaleX(-1);">
        <use xlink:href="#graph" />
      </svg>
      <div class="absolute top-0 right-0 mt-3 mr-2">
        <Tooltip>
          <template #trigger>
            <svg class="w-4 h-4 cursor-pointer">
              <use xlink:href="#info" />
            </svg>
          </template>
          <template #default>
            <div class="px-2 text-sm text-center text-white select-none">
              <div>Stats based on</div>
              <div>
                <span class="font-bold text-teal-400">{{ stats.global.count }}</span> matches
              </div>
              <div class="mt-2 text-xs italic font-normal leading-tight text-blue-100">
                Load more matches
                <br />to have better results.
              </div>
            </div>
          </template>
        </Tooltip>
      </div>
    </div>
    <div class="flex items-center py-2 mt-2">
      <div
        v-for="(role, index) in stats.role"
        :key="index"
        class="flex flex-col items-center w-1/5"
      >
        <Tooltip>
          <template #trigger>
            <div class="flex flex-col justify-end w-2 h-12 bg-blue-900 rounded-full cursor-pointer">
              <div
                :style="{height: (role.count * 3 / mostPlayedRole) * role.wins / role.count + 'rem'}"
                :class="roundedRoleWins(role.wins, role.count)"
                class="bg-green-400"
              ></div>
              <div
                :style="{height: (role.count * 3 / mostPlayedRole) * role.losses / role.count + 'rem'}"
                :class="roundedRoleLosses(role.losses, role.count)"
                class="bg-red-400"
              ></div>
            </div>
          </template>
          <template #default>
            <div class="px-2 text-sm text-center text-white select-none">
              <div>{{ role.role|capitalize }}</div>
              <span
                :class="winLossColor(role.wins, role.losses).win"
                class="font-bold"
              >{{ role.wins }}</span>
              <span class="mx-1 font-bold text-gray-400">-</span>
              <span
                :class="winLossColor(role.wins, role.losses).loss"
                class="font-bold"
              >{{ role.losses }}</span>
              <div
                :class="calculateWinrate(role.wins, role.count).color"
                class="mt-1 font-bold"
              >{{ calculateWinrate(role.wins, role.count).winrate|round }}%</div>
            </div>
          </template>
        </Tooltip>
        <div
          :style="{backgroundImage: `url(${require('@/assets/img/roles/' + role.role + '.png')})`}"
          class="w-4 h-4 mt-1 bg-center bg-cover"
        ></div>
        <div class="text-xs text-blue-200">{{ role.count }}</div>
      </div>
    </div>
    <div class="py-2 text-sm text-center">
      <div class="flex items-baseline px-4 text-xs font-semibold text-blue-300 uppercase">
        <div class="w-1/4 text-base text-left text-blue-400">Stat</div>
        <div class="w-1/4">Total</div>
        <div class="w-1/4">Per min</div>
        <div class="w-1/4">Avg</div>
      </div>
      <ul class="mt-1 text-gray-100">
        <li
          v-for="(stat, name, index) in globalStatsKeys"
          :key="index"
          :class="{'bg-blue-760': index % 2 !== 0}"
          class="flex items-center justify-between px-4 py-1 leading-tight"
        >
          <div class="w-1/4 text-left capitalize">{{ name }}</div>
          <div class="w-1/4">{{ stat }}</div>
          <div class="w-1/4">{{ stat / (stats.global.time / 60)|round }}</div>
          <div class="w-1/4">{{ stat / stats.global.count|round }}</div>
        </li>
        <li class="flex items-center justify-between px-4 py-1 leading-tight bg-blue-760">
          <div class="w-1/4 text-left whitespace-no-wrap">Time</div>
          <div class="w-1/4">{{ (stats.global.time / 3600).toFixed(1) + 'h' }}</div>
          <div class="w-1/4"></div>
          <div class="w-1/4">{{ (stats.global.time / stats.global.count)|secToTime(true) }}</div>
        </li>
        <li class="flex items-center justify-between px-4 py-1 leading-tight">
          <div class="w-1/4 text-left whitespace-no-wrap">KDA</div>
          <div
            class="w-1/4"
          >{{ (stats.global.kills + stats.global.assists) / stats.global.deaths|round }}</div>
        </li>
        <li class="flex items-center justify-between px-4 py-1 leading-tight bg-blue-760">
          <div class="w-1/4 text-left whitespace-no-wrap">Kill participation</div>
          <div class="w-1/4">{{ stats.global.kp|percent }}</div>
        </li>
      </ul>
      <template v-if="leagueStatsByType('Ranked').length">
        <div class="flex items-baseline px-4 mt-3 text-xs font-semibold text-blue-300 uppercase">
          <div class="w-2/4 text-base text-left text-blue-400">Ranked</div>
          <div class="w-1/4">Winrate</div>
          <div class="w-1/4">Record</div>
        </div>
        <ul class="mt-1 text-gray-100">
          <li
            v-for="(league, index) in leagueStatsByType('Ranked')"
            :key="index"
            :class="{'bg-blue-760': index % 2 !== 0}"
            class="flex items-center justify-between px-4 py-1 leading-tight"
          >
            <div class="w-2/4 text-left capitalize">{{ league.name.toLowerCase() }}</div>
            <div
              :class="calculateWinrate(league.wins, league.count).color"
              class="w-1/4"
            >{{ calculateWinrate(league.wins, league.count).winrate|percent }}</div>
            <div class="w-1/4">
              <span
                :class="winLossColor(league.wins, league.losses).win"
                class="font-semibold"
              >{{ league.wins }}</span>
              <span class="mx-1 font-semibold text-gray-400">-</span>
              <span
                :class="winLossColor(league.wins, league.losses).loss"
                class="font-semibold"
              >{{ league.losses }}</span>
            </div>
          </li>
        </ul>
      </template>
      <template v-if="leagueStatsByType('Normal').length">
        <div class="flex items-baseline px-4 mt-3 text-xs font-semibold text-blue-300 uppercase">
          <div class="w-2/4 text-base text-left text-blue-400">Normal</div>
          <div class="w-1/4">Winrate</div>
          <div class="w-1/4">Record</div>
        </div>
        <ul class="mt-1 text-gray-100">
          <li
            v-for="(league, index) in leagueStatsByType('Normal')"
            :key="index"
            :class="{'bg-blue-760': index % 2 !== 0}"
            class="flex items-center justify-between px-4 py-1 leading-tight"
          >
            <div class="w-2/4 text-left capitalize">{{ league.name.toLowerCase() }}</div>
            <div
              :class="calculateWinrate(league.wins, league.count).color"
              class="w-1/4"
            >{{ calculateWinrate(league.wins, league.count).winrate|percent }}</div>
            <div class="w-1/4">
              <span
                :class="winLossColor(league.wins, league.losses).win"
                class="font-semibold"
              >{{ league.wins }}</span>
              <span class="mx-1 font-semibold text-gray-400">-</span>
              <span
                :class="winLossColor(league.wins, league.losses).loss"
                class="font-semibold"
              >{{ league.losses }}</span>
            </div>
          </li>
        </ul>
      </template>

      <div class="flex items-baseline px-4 mt-3 text-xs font-semibold text-blue-300 uppercase">
        <div class="w-2/4 text-base text-left text-blue-400">Class</div>
        <div class="w-1/4">Winrate</div>
        <div class="w-1/4">Record</div>
      </div>
      <ul class="mt-1 text-gray-100">
        <li
          v-for="(championClass, index) in stats.class"
          :key="index"
          :class="{'bg-blue-760': index % 2 !== 0}"
          class="flex items-center justify-between px-4 py-1 leading-tight"
        >
          <div class="w-2/4 text-left capitalize">{{ championClass.id }}</div>
          <div
            :class="calculateWinrate(championClass.wins, championClass.count).color"
            class="w-1/4"
          >{{ calculateWinrate(championClass.wins, championClass.count).winrate|percent }}</div>
          <div class="w-1/4">
            <span
              :class="winLossColor(championClass.wins, championClass.losses).win"
              class="font-semibold"
            >{{ championClass.wins }}</span>
            <span class="mx-1 font-semibold text-gray-400">-</span>
            <span
              :class="winLossColor(championClass.wins, championClass.losses).loss"
              class="font-semibold"
            >{{ championClass.losses }}</span>
          </div>
        </li>
      </ul>
    </div>

    <div class="flex flex-col items-center pb-2 leading-snug">
      <div
        class="text-xl text-teal-400"
      >{{ calculateWinrate(stats.global.wins, stats.global.count).winrate|percent }}</div>
      <div class="flex text-sm">
        <span
          :class="winLossColor(stats.global.wins, stats.global.losses).win"
          class
        >{{ stats.global.wins }}</span>
        <span class="mx-1 font-bold text-gray-400">-</span>
        <span
          :class="winLossColor(stats.global.wins, stats.global.losses).loss"
          class
        >{{ stats.global.losses }}</span>
      </div>
      <span class="text-xs">Global winrate</span>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import Tooltip from '@/components/Common/Tooltip.vue'
import { gameModes } from '@/data/data.js'

export default {
  components: {
    Tooltip,
  },

  computed: {
    mostPlayedRole() {
      return Math.max(...this.stats.role.map(r => r.count), 0)
    },
    globalStatsKeys() {
      // eslint-disable-next-line no-unused-vars
      const { id, wins, losses, count, time, kp, ...rest } = this.stats.global
      return rest
    },
    ...mapState({
      stats: state => state.summoner.overview.stats
    }),
  },

  methods: {
    calculateWinrate(wins, count) {
      const winrate = count !== 0 ? wins / count * 100 : 0
      const color = winrate >= 50 ? 'text-green-400' : 'text-red-400'
      return {
        winrate,
        color
      }
    },
    leagueStatsByType(typeName) {
      return this.stats.league
        .map(l => {
          return { ...l, ...gameModes[l.id] }
        })
        .filter(l => l.type === typeName)
    },
    roundedRoleLosses(win, count) {
      return win === count ? 'rounded-full' : 'rounded-b-full'
    },
    roundedRoleWins(win, count) {
      return win === count ? 'rounded-full' : 'rounded-t-full'
    },
    winLossColor(win, loss) {
      const colors = {
        win: 'text-gray-200',
        loss: 'text-gray-200'
      }
      win >= loss ? colors.win = 'text-green-400' : colors.loss = 'text-red-400'
      return colors
    }
  }
}
</script>
