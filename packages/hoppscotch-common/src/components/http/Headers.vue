<template>
  <div class="flex flex-col flex-1">
    <div
      class="sticky z-10 flex items-center justify-between flex-shrink-0 pl-4 overflow-x-auto border-b bg-primary border-dividerLight top-upperMobileSecondaryStickyFold sm:top-upperSecondaryStickyFold"
    >
      <label class="font-semibold truncate text-secondaryLight">
        {{ t("request.header_list") }}
      </label>
      <div class="flex">
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          to="https://docs.hoppscotch.io/features/headers"
          blank
          :title="t('app.wiki')"
          :icon="IconHelpCircle"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="t('action.clear_all')"
          :icon="IconTrash2"
          @click="clearContent()"
        />
        <HoppButtonSecondary
          v-if="bulkMode"
          v-tippy="{ theme: 'tooltip' }"
          :title="t('state.linewrap')"
          :class="{ '!text-accent': linewrapEnabled }"
          :icon="IconWrapText"
          @click.prevent="linewrapEnabled = !linewrapEnabled"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="t('state.bulk_mode')"
          :icon="IconEdit"
          :class="{ '!text-accent': bulkMode }"
          @click="bulkMode = !bulkMode"
        />
        <HoppButtonSecondary
          v-tippy="{ theme: 'tooltip' }"
          :title="t('add.new')"
          :icon="IconPlus"
          :disabled="bulkMode"
          @click="addHeader"
        />
      </div>
    </div>
    <div v-if="bulkMode" ref="bulkEditor" class="flex flex-col flex-1"></div>
    <div v-else>
      <draggable
        v-model="workingHeaders"
        :item-key="(header) => `header-${header.id}`"
        animation="250"
        handle=".draggable-handle"
        draggable=".draggable-content"
        ghost-class="cursor-move"
        chosen-class="bg-primaryLight"
        drag-class="cursor-grabbing"
      >
        <template #item="{ element: header, index }">
          <div
            class="flex border-b divide-x divide-dividerLight border-dividerLight draggable-content group"
          >
            <span>
              <HoppButtonSecondary
                v-tippy="{
                  theme: 'tooltip',
                  delay: [500, 20],
                  content:
                    index !== workingHeaders?.length - 1
                      ? t('action.drag_to_reorder')
                      : null,
                }"
                :icon="IconGripVertical"
                class="cursor-auto text-primary hover:text-primary"
                :class="{
                  'draggable-handle group-hover:text-secondaryLight !cursor-grab':
                    index !== workingHeaders?.length - 1,
                }"
                tabindex="-1"
              />
            </span>
            <HoppSmartAutoComplete
              :placeholder="`${t('count.header', { count: index + 1 })}`"
              :source="commonHeaders"
              :spellcheck="false"
              :value="header.key"
              autofocus
              styles=" bg-transparent flex flex-1
            py-1 px-4 truncate "
              class="flex-1 !flex"
              @input="
                updateHeader(index, {
                  id: header.id,
                  key: $event,
                  value: header.value,
                  active: header.active,
                })
              "
            />
            <SmartEnvInput
              v-model="header.value"
              :placeholder="`${t('count.value', { count: index + 1 })}`"
              @change="
                updateHeader(index, {
                  id: header.id,
                  key: header.key,
                  value: $event,
                  active: header.active,
                })
              "
            />
            <span>
              <HoppButtonSecondary
                v-tippy="{ theme: 'tooltip' }"
                :title="
                  header.hasOwnProperty('active')
                    ? header.active
                      ? t('action.turn_off')
                      : t('action.turn_on')
                    : t('action.turn_off')
                "
                :icon="
                  header.hasOwnProperty('active')
                    ? header.active
                      ? IconCheckCircle
                      : IconCircle
                    : IconCheckCircle
                "
                color="green"
                @click="
                  updateHeader(index, {
                    id: header.id,
                    key: header.key,
                    value: header.value,
                    active: !header.active,
                  })
                "
              />
            </span>
            <span>
              <HoppButtonSecondary
                v-tippy="{ theme: 'tooltip' }"
                :title="t('action.remove')"
                :icon="IconTrash"
                color="red"
                @click="deleteHeader(index)"
              />
            </span>
          </div>
        </template>
      </draggable>

      <draggable
        v-model="computedHeaders"
        item-key="id"
        animation="250"
        handle=".draggable-handle"
        draggable=".draggable-content"
        ghost-class="cursor-move"
        chosen-class="bg-primaryLight"
        drag-class="cursor-grabbing"
      >
        <template #item="{ element: header, index }">
          <div
            class="flex border-b divide-x divide-dividerLight border-dividerLight draggable-content group"
          >
            <span>
              <HoppButtonSecondary
                :icon="IconLock"
                class="opacity-25 cursor-auto text-secondaryLight bg-divider"
                tabindex="-1"
              />
            </span>
            <SmartEnvInput
              v-model="header.header.key"
              :placeholder="`${t('count.value', { count: index + 1 })}`"
              readonly
            />
            <SmartEnvInput
              :model-value="mask(header)"
              :placeholder="`${t('count.value', { count: index + 1 })}`"
              readonly
            />
            <span>
              <HoppButtonSecondary
                v-if="header.source === 'auth'"
                :icon="masking ? IconEye : IconEyeOff"
                @click="toggleMask()"
              />
              <HoppButtonSecondary
                v-else
                :icon="IconArrowUpRight"
                class="cursor-auto text-primary hover:text-primary"
              />
            </span>
            <span>
              <HoppButtonSecondary
                :icon="IconArrowUpRight"
                @click="changeTab(header.source)"
              />
            </span>
          </div>
        </template>
      </draggable>
      <div
        v-if="workingHeaders.length === 0"
        class="flex flex-col items-center justify-center p-4 text-secondaryLight"
      >
        <img
          :src="`/images/states/${colorMode.value}/add_category.svg`"
          loading="lazy"
          class="inline-flex flex-col object-contain object-center w-16 h-16 my-4"
          :alt="`${t('empty.headers')}`"
        />
        <span class="pb-4 text-center">{{ t("empty.headers") }}</span>
        <HoppButtonSecondary
          filled
          :label="`${t('add.new')}`"
          :icon="IconPlus"
          class="mb-4"
          @click="addHeader"
        />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import IconHelpCircle from "~icons/lucide/help-circle"
import IconTrash2 from "~icons/lucide/trash-2"
import IconEdit from "~icons/lucide/edit"
import IconPlus from "~icons/lucide/plus"
import IconGripVertical from "~icons/lucide/grip-vertical"
import IconCheckCircle from "~icons/lucide/check-circle"
import IconCircle from "~icons/lucide/circle"
import IconTrash from "~icons/lucide/trash"
import IconLock from "~icons/lucide/lock"
import IconEye from "~icons/lucide/eye"
import IconEyeOff from "~icons/lucide/eye-off"
import IconArrowUpRight from "~icons/lucide/arrow-up-right"
import IconWrapText from "~icons/lucide/wrap-text"
import { useColorMode } from "@composables/theming"
import { computed, reactive, Ref, ref, watch } from "vue"
import { isEqual, cloneDeep } from "lodash-es"
import {
  HoppRESTHeader,
  parseRawKeyValueEntriesE,
  rawKeyValueEntriesToString,
  RawKeyValueEntry,
} from "@hoppscotch/data"
import { flow, pipe } from "fp-ts/function"
import * as RA from "fp-ts/ReadonlyArray"
import * as E from "fp-ts/Either"
import * as O from "fp-ts/Option"
import * as A from "fp-ts/Array"
import draggable from "vuedraggable-es"
import { RequestOptionTabs } from "./RequestOptions.vue"
import { useCodemirror } from "@composables/codemirror"
import {
  getRESTRequest,
  restHeaders$,
  restRequest$,
  setRESTHeaders,
} from "~/newstore/RESTSession"
import { commonHeaders } from "~/helpers/headers"
import { useI18n } from "@composables/i18n"
import { useReadonlyStream, useStream } from "@composables/stream"
import { useToast } from "@composables/toast"
import linter from "~/helpers/editor/linting/rawKeyValue"
import { throwError } from "~/helpers/functional/error"
import { objRemoveKey } from "~/helpers/functional/object"
import {
  ComputedHeader,
  getComputedHeaders,
} from "~/helpers/utils/EffectiveURL"
import { aggregateEnvs$, getAggregateEnvs } from "~/newstore/environments"

const t = useI18n()
const toast = useToast()
const colorMode = useColorMode()

const idTicker = ref(0)

const bulkMode = ref(false)
const bulkHeaders = ref("")
const bulkEditor = ref<any | null>(null)
const linewrapEnabled = ref(true)

const deletionToast = ref<{ goAway: (delay: number) => void } | null>(null)

const emit = defineEmits<{
  (e: "change-tab", value: RequestOptionTabs): void
}>()

useCodemirror(
  bulkEditor,
  bulkHeaders,
  reactive({
    extendedEditorConfig: {
      mode: "text/x-yaml",
      placeholder: `${t("state.bulk_mode_placeholder")}`,
      lineWrapping: linewrapEnabled,
    },
    linter,
    completer: null,
    environmentHighlights: true,
  })
)

// The functional headers list (the headers actually in the system)
const headers = useStream(restHeaders$, [], setRESTHeaders) as Ref<
  HoppRESTHeader[]
>

// The UI representation of the headers list (has the empty end headers)
const workingHeaders = ref<Array<HoppRESTHeader & { id: number }>>([
  {
    id: idTicker.value++,
    key: "",
    value: "",
    active: true,
  },
])

// Rule: Working Headers always have last element is always an empty header
watch(workingHeaders, (headersList) => {
  if (
    headersList.length > 0 &&
    headersList[headersList.length - 1].key !== ""
  ) {
    workingHeaders.value.push({
      id: idTicker.value++,
      key: "",
      value: "",
      active: true,
    })
  }
})

// Sync logic between headers and working/bulk headers
watch(
  headers,
  (newHeadersList) => {
    // Sync should overwrite working headers
    const filteredWorkingHeaders = pipe(
      workingHeaders.value,
      A.filterMap(
        flow(
          O.fromPredicate((e) => e.key !== ""),
          O.map(objRemoveKey("id"))
        )
      )
    )

    const filteredBulkHeaders = pipe(
      parseRawKeyValueEntriesE(bulkHeaders.value),
      E.map(
        flow(
          RA.filter((e) => e.key !== ""),
          RA.toArray
        )
      ),
      E.getOrElse(() => [] as RawKeyValueEntry[])
    )

    if (!isEqual(newHeadersList, filteredWorkingHeaders)) {
      workingHeaders.value = pipe(
        newHeadersList,
        A.map((x) => ({ id: idTicker.value++, ...x }))
      )
    }

    if (!isEqual(newHeadersList, filteredBulkHeaders)) {
      bulkHeaders.value = rawKeyValueEntriesToString(newHeadersList)
    }
  },
  { immediate: true }
)

watch(workingHeaders, (newWorkingHeaders) => {
  const fixedHeaders = pipe(
    newWorkingHeaders,
    A.filterMap(
      flow(
        O.fromPredicate((e) => e.key !== ""),
        O.map(objRemoveKey("id"))
      )
    )
  )

  if (!isEqual(headers.value, fixedHeaders)) {
    headers.value = cloneDeep(fixedHeaders)
  }
})

watch(bulkHeaders, (newBulkHeaders) => {
  const filteredBulkHeaders = pipe(
    parseRawKeyValueEntriesE(newBulkHeaders),
    E.map(
      flow(
        RA.filter((e) => e.key !== ""),
        RA.toArray
      )
    ),
    E.getOrElse(() => [] as RawKeyValueEntry[])
  )

  if (!isEqual(headers.value, filteredBulkHeaders)) {
    headers.value = filteredBulkHeaders
  }
})

const addHeader = () => {
  workingHeaders.value.push({
    id: idTicker.value++,
    key: "",
    value: "",
    active: true,
  })
}

const updateHeader = (
  index: number,
  header: HoppRESTHeader & { id: number }
) => {
  workingHeaders.value = workingHeaders.value.map((h, i) =>
    i === index ? header : h
  )
}

const deleteHeader = (index: number) => {
  const headersBeforeDeletion = cloneDeep(workingHeaders.value)

  if (
    !(
      headersBeforeDeletion.length > 0 &&
      index === headersBeforeDeletion.length - 1
    )
  ) {
    if (deletionToast.value) {
      deletionToast.value.goAway(0)
      deletionToast.value = null
    }

    deletionToast.value = toast.success(`${t("state.deleted")}`, {
      action: [
        {
          text: `${t("action.undo")}`,
          onClick: (_, toastObject) => {
            workingHeaders.value = headersBeforeDeletion
            toastObject.goAway(0)
            deletionToast.value = null
          },
        },
      ],

      onComplete: () => {
        deletionToast.value = null
      },
    })
  }

  workingHeaders.value = pipe(
    workingHeaders.value,
    A.deleteAt(index),
    O.getOrElseW(() => throwError("Working Headers Deletion Out of Bounds"))
  )
}

const clearContent = () => {
  // set params list to the initial state
  workingHeaders.value = [
    {
      id: idTicker.value++,
      key: "",
      value: "",
      active: true,
    },
  ]

  bulkHeaders.value = ""
}

const restRequest = useReadonlyStream(restRequest$, getRESTRequest())
const aggregateEnvs = useReadonlyStream(aggregateEnvs$, getAggregateEnvs())

const computedHeaders = computed(() =>
  getComputedHeaders(restRequest.value, aggregateEnvs.value).map(
    (header, index) => ({
      id: `header-${index}`,
      ...header,
    })
  )
)

const masking = ref(true)

const toggleMask = () => {
  masking.value = !masking.value
}

const mask = (header: ComputedHeader) => {
  if (header.source === "auth" && masking.value)
    return header.header.value.replace(/\S/gi, "*")
  return header.header.value
}

const changeTab = (tab: ComputedHeader["source"]) => {
  if (tab === "auth") emit("change-tab", "authorization")
  else emit("change-tab", "bodyParams")
}
</script>
