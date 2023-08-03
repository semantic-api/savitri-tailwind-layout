<script setup lang="ts">
import { type MenuSchema, useRouter, useNavbar } from '@savitri/web'
import { inject, reactive, toRefs, computed, onMounted } from 'vue'
import { daysAgo } from '@semantic-api/common'
import { SvInfo, SvIcon, SvPicture } from '@savitri/ui'

const menuSchema = inject<MenuSchema>('menuSchema')

const navbarRefs = reactive({
  routesWithChildren: [],
  isCurrent: (..._args: Parameters<Awaited<ReturnType<typeof useNavbar>>['isCurrent']>) => false
})

const {
  routesWithChildren,
  isCurrent

} = toRefs(navbarRefs)

const router = await useRouter()
const push = (...args: Parameters<typeof router.push>) => {
  window.scrollTo(0, 0)
  router.push(...args)
}

onMounted(async () => {
  /*useStore('user').functions.ping(null, {
    skipLoading: true
  })
  */

  const navbar = await useNavbar({ schema: menuSchema })
  Object.assign(navbarRefs, navbar)
})

const expandedIndex = ref(0)
const expandedMenu = computed(() => routesWithChildren.value[expandedIndex.value])

const logoUrl = new URL('/static/logo.png', import.meta.url).href
</script>

<template>
  <div class="
    tw-flex
  ">
    <nav class="
      tw-sticky
      tw-inset-0
      tw-h-screen
      tw-flex
      tw-overflow-hidden
      tw-border-r
      tw-bg-gray-50
      lg:tw-w-[20rem]
    ">
      <div class="
        tw-flex
        tw-flex-col
        tw-border-r
        tw-p-4
      ">
        <img
          v-clickable
          :src="logoUrl"
          class="
            tw-h-16
            tw-w-16
            tw-object-contain
            tw-mb-6
          "
          @click="push('/dashboard')"
        />
        <div class="
          tw-flex
          tw-flex-col
          tw-items-center
          tw-gap-4
          tw-mb-auto
        ">
          <sv-info
            where="right"
            v-for="(item, index) in routesWithChildren"
            :key="`item-${index}`"
          >
            <template #text>
              {{ item.meta.title }}
            </template>
            <sv-icon
              v-clickable
              :class="`
                tw-p-4
                tw-border
                tw-rounded-xl
                tw-justify-center
                hover:tw-bg-gray-200
                ${index === expandedIndex && 'tw-bg-gray-100'}
              `"

              :name="item.meta.icon"
              @click="expandedIndex = index; router.push({ name: expandedMenu.children[0].name })"
            ></sv-icon>
          </sv-info>

        </div>

        <sv-picture
          v-clickable
          :file-id="currentUser.picture?._id || currentUser.picture"
          class="
            tw-rounded-full
            tw-overflow-hidden
            tw-border
            tw-w-16
            tw-h-16
          "

          @click="$router.push('/dashboard/user/profile')"
        ></sv-picture>
      </div>

      <div
        v-if="expandedMenu"
        class="
          tw-flex
          tw-flex-col
          tw-flex-1
          tw-p-4
        "
      >
        <div class="
          tw-p-4
          tw-font-bold
          tw-text-lg
        ">
          {{ expandedMenu.meta.title }}
        </div>

        <div class="
          tw-flex
          tw-flex-col
          tw-gap-2
        ">
          <sv-icon
            v-clickable
            v-for="(subitem, index) in expandedMenu.children"
            :key="`subitem-${index}`"
            :name="subitem.meta.icon"

            :class="`
              tw-font-[300]
              tw-text-[10pt]
              tw-rounded-xl
              tw-p-4
              hover:tw-bg-gray-200
              ${isCurrent(subitem) && 'tw-bg-gray-100'}
            `"

            @click="$router.push({ name: subitem.name })"
          >
            {{ capitalize(subitem.meta.title) }}
          </sv-icon>
        </div>
      </div>

    </nav>

    <div class="
      tw-flex
      tw-flex-col
      tw-flex-1
    ">
      <div class="
        tw-sticky
        tw-inset-0
        tw-flex
        tw-items-center
        tw-gap-6
        tw-bg-white
        tw-p-6
        tw-z-20
      ">
        <sv-icon
          :name="viewIcon"
          class="
            tw-text-xl
          "
        >
          {{ capitalize(viewTitle) }}
        </sv-icon>

        <router-view name="topbar"></router-view>
      </div>

      <div class="
        tw-flex
        tw-flex-col
        tw-gap-[2rem]
        tw-p-6
      ">
        <router-view></router-view>
      </div>
    </div>
  </div>

</template>
