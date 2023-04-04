<template>
  <div>
    <HoppSmartTabs
      :id="'environments_tab'"
      v-model="selectedEnvironmentTab"
      render-inactive-tabs
    >
      <HoppSmartTab
        :id="'my-environments'"
        :label="`${t('environment.my_environments')}`"
      />
    </HoppSmartTabs>
  </div>
</template>

<script setup lang="ts">
import { nextTick, ref, watch } from "vue"
import { GetMyTeamsQuery } from "~/helpers/backend/graphql"
import { onLoggedIn } from "@composables/auth"
import { platform } from "~/platform"
import TeamListAdapter from "~/helpers/teams/TeamListAdapter"
import { useReadonlyStream } from "@composables/stream"
import { useLocalState } from "~/newstore/localstate"
import { useI18n } from "@composables/i18n"
import IconDone from "~icons/lucide/check"
import IconUsers from "~icons/lucide/users"
import { invokeAction } from "~/helpers/actions"

const t = useI18n()

type TeamData = GetMyTeamsQuery["myTeams"][number]

type SelectedTeam = TeamData | undefined

type EnvironmentTabs = "my-environments" | "team-environments"

// Template refs
const tippyActions = ref<any | null>(null)
const selectedEnvironmentTab = ref<EnvironmentTabs>("my-environments")

defineProps<{
  environmentType: {
    type: "my-environments" | "team-environments"
    selectedTeam: SelectedTeam
  }
}>()

const emit = defineEmits<{
  (e: "update-environment-type", tabID: EnvironmentTabs): void
  (e: "update-selected-team", team: SelectedTeam): void
}>()

const currentUser = useReadonlyStream(
  platform.auth.getCurrentUserStream(),
  platform.auth.getCurrentUser()
)

const adapter = new TeamListAdapter(true)
const myTeams = useReadonlyStream(adapter.teamList$, null)
const REMEMBERED_TEAM_ID = useLocalState("REMEMBERED_TEAM_ID")
let teamListFetched = false

watch(myTeams, (teams) => {
  if (teams && !teamListFetched) {
    teamListFetched = true
    if (REMEMBERED_TEAM_ID.value && currentUser.value) {
      const team = teams.find((t) => t.id === REMEMBERED_TEAM_ID.value)
      if (team) updateSelectedTeam(team)
    }
  }
})

watch(
  () => currentUser.value,
  (user) => {
    if (!user) {
      selectedEnvironmentTab.value = "my-environments"
    }
  }
)

onLoggedIn(() => {
  try {
    adapter.initialize()
  } catch (e) {}
})

const onTeamSelectIntersect = () => {
  // Load team data as soon as intersection
  adapter.fetchList()
}

const updateEnvironmentType = (tabID: EnvironmentTabs) => {
  emit("update-environment-type", tabID)
}

const updateSelectedTeam = (team: SelectedTeam) => {
  REMEMBERED_TEAM_ID.value = team?.id
  emit("update-selected-team", team)
}

watch(selectedEnvironmentTab, (newValue: EnvironmentTabs) => {
  if (newValue === "team-environments" && !currentUser.value) {
    invokeAction("modals.login.toggle")
    nextTick(() => (selectedEnvironmentTab.value = "my-environments"))
  } else updateEnvironmentType(newValue)
})
</script>
