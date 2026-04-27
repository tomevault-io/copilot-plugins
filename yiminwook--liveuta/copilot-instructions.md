## liveuta

> ├── .cursor                      # cusror rules 폴더

## 프로젝트 구조: 주요 폴더 구조예시
project-root/
│
├── .cursor                      # cusror rules 폴더
├── .vscode                      # vsCode setting 폴더
├── analyze                      # next-bundle-analyzer 결과 폴더
├── e2e                          # e2e - playwright 테스트 코드 폴더
├── messages                     # i18next 번역 json 폴더
│   ├── en.json
│   ├── ja.json
│   ├── ko.json
│   └── README.md
├── public                       # public static assets 폴더
│   ├── assets
│   │   └── meta-image.png
│   ├── mockServiceWorker.js
│   ├── sw.js
│   └── theme.css
├── src           
│   ├── apis                       # client and server side fetch 함수 폴더
│   │   ├── cached.ts              # react.cache 캐시 모음
│   │   ├── fetcher.ts             # ky 라이브러리 fetch 함수 모음  
│   │   └── getQueryClient.ts      # react-query client
│   ├── app                        # app router 폴더
│   │   ├── [locale]               # 일반 유저들이 접근가능한 페이지 폴더
│   │   │   ├── [...catchAll]      # not-found catch all segments
│   │   │   │   └── page.tsx
│   │   │   ├── sentry-test
│   │   │   │   └── page.tsx
│   │   │   ├── forbidden.tsx   
│   │   │   ├── layout.tsx
│   │   │   ├── not-found.tsx
│   │   │   └── unauthorized.tsx
│   │   ├── admin                  # 관리자 페이지 폴더
│   │   │   ├── layout.tsx
│   │   │   └── page.tsx
│   │   │       
│   │   ├── api                   # api route handler 폴더
│   │   │   ├── auth
│   │   │   │   └── [...nextauth] # next-auth route handlers
│   │   │   │       └── route.ts
│   │   │   └── v1                # api version 
│   │   │       ├── blacklist
│   │   │       │   └── route.ts
│   │   │       ├── channel
│   │   │       │   └── route.ts
│   │   │       ├── featured
│   │   │       │   └── route.ts
│   │   │       ├── member
│   │   │       │   └── route.ts
│   │   │       ├── metadata
│   │   │       │   └── route.ts
│   │   │       ├── revalidate
│   │   │       │   └── route.ts
│   │   │       ├── schedule
│   │   │       │   └── route.ts
│   │   │       ├── setlist
│   │   │       │   └── route.ts
│   │   │       ├── whitelist
│   │   │       │   └── route.ts
│   │   │       └── youtube-channel
│   │   │           └── route.ts
│   │   ├── favicon.ico
│   │   ├── global-error.tsx
│   │   ├── layout.tsx
│   │   ├── manifest.ts
│   │   ├── page.tsx
│   │   ├── robots.ts
│   │   └── sitemap.ts
│   ├── assets                               # private static assets 폴더
│   │   └── image
│   ├── components                           # 컴포넌트 폴더 도메인 별로 구분
│   │   ├── common                           # 공통 컴포넌트 폴더
│   │   │   ├── authorization                # client side 세션관리 컴포넌트 폴더
│   │   │   │   ├── Administrator.tsx
│   │   │   │   ├── Authorized.tsx
│   │   │   │   ├── UnAuthorized.tsx
│   │   │   │   └── withSession.tsx
│   │   │   ├── button
│   │   │   │   ├── ClearButton.tsx
│   │   │   │   ├── CopyButton.tsx
│   │   │   │   ├── HamburgerBtn.module.scss
│   │   │   │   ├── HamburgerBtn.tsx
│   │   │   │   ├── MoreButton.module.scss
│   │   │   │   ├── MoreButton.tsx
│   │   │   │   ├── PasteButton.tsx
│   │   │   │   └── ToggleButton.tsx
│   │   │   ├── channelCard
│   │   │   │   ├── ChannelCard.module.scss
│   │   │   │   └── ChannelCard.tsx
│   │   │   ├── command
│   │   │   │   ├── CommandMenu.module.scss
│   │   │   │   ├── CommandMenu.tsx
│   │   │   │   ├── Context.tsx
│   │   │   │   └── GlobalCmd.tsx
│   │   │   ├── DataFetchingObserver.tsx
│   │   │   ├── Footer.module.scss
│   │   │   ├── Footer.tsx
│   │   │   ├── header
│   │   │   │   ├── DesktopNav.tsx
│   │   │   │   ├── Header.module.scss
│   │   │   │   ├── Header.tsx
│   │   │   │   ├── HeaderMenu.module.scss
│   │   │   │   └── HeaderMenu.tsx
│   │   │   ├── Iframe.module.scss
│   │   │   ├── Iframe.tsx
│   │   │   ├── input
│   │   │   │   ├── SearchInput.module.scss
│   │   │   │   └── SearchInput.tsx
│   │   │   ├── loading
│   │   │   │   ├── GlobalLoading.module.scss
│   │   │   │   ├── Loading.module.scss
│   │   │   │   ├── loading.tsx
│   │   │   │   ├── MainLoading.tsx
│   │   │   │   ├── RingLoader.tsx
│   │   │   │   ├── SquareToRound.tsx
│   │   │   │   └── Wave.tsx
│   │   │   ├── modal
│   │   │   │   ├── AlertModal.tsx
│   │   │   │   ├── ChannelCardModal.module.scss
│   │   │   │   ├── ChannelCardModal.tsx
│   │   │   │   ├── ConfirmModal.tsx
│   │   │   │   ├── ErrorModal.tsx
│   │   │   │   ├── Modal.module.scss
│   │   │   │   └── Modal.tsx
│   │   │   ├── Nodata.module.scss
│   │   │   ├── Nodata.tsx
│   │   │   ├── PageView.tsx
│   │   │   ├── player
│   │   │   │   ├── DndComponents.module.scss
│   │   │   │   ├── DndComponents.tsx
│   │   │   │   ├── GlobalPip.tsx
│   │   │   │   ├── LiveChat.tsx
│   │   │   │   ├── LiveChatPlayer.tsx
│   │   │   │   ├── Player.module.scss
│   │   │   │   ├── PlayerBase.tsx
│   │   │   │   └── PlayerPlaceholder.tsx
│   │   │   ├── scheduleCard
│   │   │   │   ├── Card.module.scss
│   │   │   │   ├── Card.tsx
│   │   │   │   ├── CardDesc.tsx
│   │   │   │   ├── CardImage.tsx
│   │   │   │   ├── CardMenu.module.scss
│   │   │   │   ├── CardMenu.tsx
│   │   │   │   ├── CardStatus.tsx
│   │   │   │   ├── CardViewer.tsx
│   │   │   │   ├── ScheduleCardSkeleton.module.scss
│   │   │   │   ├── ScheduleCardSkeleton.tsx
│   │   │   │   ├── SliderCard.module.scss
│   │   │   │   ├── SliderCard.tsx
│   │   │   │   └── SliderCardSkeleton.tsx
│   │   │   ├── sidebar
│   │   │   │   ├── Account.tsx
│   │   │   │   ├── ExternalLinksSection.tsx
│   │   │   │   ├── IndexSection.tsx
│   │   │   │   ├── NavItem.tsx
│   │   │   │   └── Sidebar.module.scss
│   │   │   ├── TimestampText.module.scss
│   │   │   ├── TimestampText.tsx
│   │   │   ├── utils
│   │   │   │   ├── For.tsx
│   │   │   │   ├── Show.tsx
│   │   │   │   └── Switch.tsx
│   │   │   ├── Vaul.module.scss
│   │   │   └── Vaul.tsx
│   │   ├── config
│   │   │   ├── AppProvider.tsx
│   │   │   ├── DefaultHead.tsx
│   │   │   ├── Devtools.tsx
│   │   │   ├── Document.tsx
│   │   │   ├── GlobalScrollbar.tsx
│   │   │   ├── GoogleAnalytics.tsx
│   │   │   ├── GoogleTagManager.tsx
│   │   │   ├── Hotkeys.tsx
│   │   │   ├── index.tsx
│   │   │   ├── MantineProvider.tsx
│   │   │   ├── ModalContainer.tsx
│   │   │   ├── NextAuth.tsx
│   │   │   ├── NProgress.tsx
│   │   │   ├── Particle.tsx
│   │   │   ├── ReactQuery.tsx
│   │   │   ├── ServiceWorker.tsx
│   │   │   ├── ThemeScript.tsx
│   │   │   ├── ToastBox.module.scss
│   │   │   └── ToastBox.tsx
│   │   ├── dev
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── PostBox.tsx
│   │   │   └── TokenBox.tsx
│   │   ├── featured
│   │   │   ├── RanckingTable.tsx
│   │   │   ├── RankingTable.module.scss
│   │   │   └── RankingTable.tsx
│   │   ├── home
│   │   │   ├── ChannelSlider.module.scss
│   │   │   ├── ChannelSlider.tsx
│   │   │   ├── ScheduleSlider.module.scss
│   │   │   └── ScheduleSlider.tsx
│   │   ├── login
│   │   │   ├── Home.module.scss
│   │   │   └── Home.tsx
│   │   ├── multi
│   │   │   ├── grid
│   │   │   │   ├── Grid.module.scss
│   │   │   │   ├── GridCore.tsx
│   │   │   │   ├── GridItem.tsx
│   │   │   │   ├── GridItemHandler.tsx
│   │   │   │   ├── GridNav.module.scss
│   │   │   │   ├── GridNav.tsx
│   │   │   │   ├── GridNavItem.tsx
│   │   │   │   ├── GridPlayer.tsx
│   │   │   │   ├── helper.ts
│   │   │   │   └── index.tsx
│   │   │   ├── Home.module.scss
│   │   │   └── Home.tsx
│   │   ├── my
│   │   │   ├── Blacklist.tsx
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── List.module.scss
│   │   │   ├── ListPlaceholder.tsx
│   │   │   └── Whitelist.tsx
│   │   ├── schedule
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── MobileNavButton.tsx
│   │   │   ├── NavTab.tsx
│   │   │   ├── QueryBtn.module.scss
│   │   │   ├── QueryBtn.tsx
│   │   │   ├── ScheduleNav.module.scss
│   │   │   ├── ScheduleNav.tsx
│   │   │   ├── ScheduleNavModal.module.scss
│   │   │   ├── ScheduleNavModal.tsx
│   │   │   ├── ScheduleSection.module.scss
│   │   │   ├── ScheduleSection.tsx
│   │   │   ├── ToggleFavorite.tsx
│   │   │   ├── TopBtn.tsx
│   │   │   ├── TopSection.module.scss
│   │   │   ├── TopSection.tsx
│   │   │   ├── VideoTypeRadio.tsx
│   │   │   └── VideoTypeSelect.tsx
│   │   ├── setlist
│   │   │   ├── Drawer.module.scss
│   │   │   ├── Drawer.tsx
│   │   │   ├── DrawerContext.tsx
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── Nav.module.scss
│   │   │   ├── Nav.tsx
│   │   │   ├── PostDrawer.module.scss
│   │   │   ├── PostDrawer.tsx
│   │   │   ├── PostForm.module.scss
│   │   │   ├── PostForm.tsx
│   │   │   ├── Row.tsx
│   │   │   ├── SearchForm.module.scss
│   │   │   ├── SearchForm.tsx
│   │   │   ├── Table.module.scss
│   │   │   └── Table.tsx
│   │   ├── setlistCreate
│   │   │   ├── Context.tsx
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── Player.module.scss
│   │   │   ├── Player.tsx
│   │   │   ├── SetlistControlSection.module.scss
│   │   │   ├── SetlistControlSection.tsx
│   │   │   ├── Table.module.scss
│   │   │   ├── Table.tsx
│   │   │   ├── TableBody.tsx
│   │   │   ├── TableHeadActions.tsx
│   │   │   ├── TableRow.tsx
│   │   │   └── UrlInput.tsx
│   │   ├── setlistId
│   │   │   ├── Desc.module.scss
│   │   │   ├── Desc.tsx
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── Info.module.scss
│   │   │   ├── Info.tsx
│   │   │   └── SetlistPlayer.tsx
│   │   ├── setting
│   │   │   ├── AutoSync.module.scss
│   │   │   ├── AutoSync.tsx
│   │   │   ├── Home.module.scss
│   │   │   ├── Home.tsx
│   │   │   ├── LanguageSelect.module.scss
│   │   │   ├── LanguageSelect.tsx
│   │   │   ├── Setting.module.scss
│   │   │   ├── ThemeSelect.module.scss
│   │   │   └── ThemeSelect.tsx
│   │   └── utils
│   │       ├── common
│   │       │   ├── Breadcrumb.tsx
│   │       │   ├── BreadcrumbContext.tsx
│   │       │   ├── Config.module.scss
│   │       │   ├── Config.tsx
│   │       │   ├── Header.module.scss
│   │       │   ├── Header.tsx
│   │       │   ├── Links.tsx
│   │       │   ├── MobileDrawer.module.scss
│   │       │   ├── MobileDrawer.tsx
│   │       │   ├── MobileDrawerContext.tsx
│   │       │   ├── Sidebar.module.scss
│   │       │   └── Sidebar.tsx
│   │       ├── converters
│   │       │   └── base64
│   │       │       ├── Context.tsx
│   │       │       ├── convert.ts
│   │       │       ├── Home.module.scss
│   │       │       └── Home.tsx
│   │       └── youtube
│   │           └── thumbnail
│   │               ├── Home.module.scss
│   │               └── Home.tsx
│   ├── constants                        # app 설정값 관련 폴더
│   │   ├── index.ts
│   │   ├── metaData.ts
│   │   ├── multi.ts
│   │   ├── pip.ts
│   │   ├── revalidateTag.ts
│   │   └── siteConfig.ts
│   ├── hooks                            # 커스텀훅 폴더 
│   │   ├── useCachedData.ts
│   │   ├── useDebounce.ts
│   │   ├── useDeleteBlacklist.ts
│   │   ├── useDeleteWhitelist.ts
│   │   ├── useInfiniteScheduleData.ts
│   │   ├── useLocation.ts
│   │   ├── usePostBlacklist.ts
│   │   ├── usePostWhitelist.ts
│   │   ├── useRender.ts
│   │   ├── useReservePush.ts
│   │   ├── useSchedule.ts
│   │   ├── useScheduleStatus.ts
│   │   ├── useStopPropagation.ts
│   │   ├── useStorage.ts
│   │   └── useTransition.ts
│   ├── instrumentation.ts                     # sentry-server 설정 파일
│   ├── libraries                              # thrid-party 라이브러리 관련 파일
│   │   ├── cloudflare
│   │   │   └── worker.js
│   │   ├── dayjs.ts                           # dayjs 설정 파일
│   │   ├── error                              # error 정의 폴더 
│   │   │   ├── badRequestError.ts
│   │   │   ├── clientError.ts
│   │   │   ├── customServerError.ts
│   │   │   ├── Fire.tsx
│   │   │   └── handler.ts
│   │   ├── firebase
│   │   │   ├── admin.ts
│   │   │   ├── client.ts
│   │   │   ├── firebaseClient.json
│   │   │   └── generateFcmToken.ts
│   │   ├── framer
│   │   │   ├── index.ts
│   │   │   └── Motion.tsx
│   │   ├── i18n
│   │   │   ├── client.tsx
│   │   │   ├── config.ts
│   │   │   ├── i18next.ts
│   │   │   ├── index.tsx
│   │   │   ├── middleware.ts
│   │   │   ├── README.md
│   │   │   ├── server.tsx
│   │   │   └── type.ts
│   │   ├── mongoDB
│   │   │   ├── channels.ts
│   │   │   └── index.ts
│   │   ├── nextAuth
│   │   │   └── index.ts
│   │   ├── oracleDB
│   │   │   ├── auth
│   │   │   │   ├── service.ts
│   │   │   │   └── sql.ts
│   │   │   ├── blacklist
│   │   │   │   ├── service.ts
│   │   │   │   └── sql.ts
│   │   │   ├── connection.ts
│   │   │   ├── metadata
│   │   │   │   ├── service.ts
│   │   │   │   └── sql.ts
│   │   │   ├── setlist
│   │   │   │   ├── service.ts
│   │   │   │   └── sql.ts
│   │   │   ├── table.ts
│   │   │   └── whitelist
│   │   │       ├── service.ts
│   │   │       └── sql.ts
│   │   ├── youtube
│   │   │   ├── index.ts
│   │   │   ├── url.test.ts
│   │   │   └── url.ts
│   │   └── zustand
│   │       ├── customStorage.ts
│   │       └── useStorageDOMEvents.ts
│   ├── mocks
│   │   ├── browser.ts
│   │   ├── enableServer.ts
│   │   ├── handlers
│   │   │   └── index.ts
│   │   ├── msw.test.ts
│   │   ├── MswProvider.tsx
│   │   └── server.ts
│   ├── stores                           # 전역 상태관리 폴더 도메인 별로 구분
│   │   ├── app.ts
│   │   ├── modal.ts
│   │   ├── multiView.ts
│   │   └── player.ts
│   ├── styles                           # global, thrid-party scss 폴더
│   │   ├── _placeholder.scss
│   │   ├── _util.scss
│   │   ├── _var.scss
│   │   ├── global.scss
│   │   ├── mantine
│   │   │   ├── core.scss
│   │   │   ├── dates.scss
│   │   │   └── theme.scss
│   │   ├── README.md
│   │   ├── reset.css
│   │   ├── swiper
│   │   │   ├── core.scss
│   │   │   ├── navigation.scss
│   │   │   ├── pagination.scss
│   │   │   └── scrollbar.scss
│   │   ├── theme.ts
│   │   └── variable.module.scss
│   ├── temps                                     # 폐기된 파일 임시보관 폴더
│   │   ├── push
│   │   │   ├── google
│   │   │   │   └── route.ts
│   │   │   ├── http
│   │   │   │   └── route.ts
│   │   │   └── reserve
│   │   │       └── route.ts
│   │   ├── scrollmagicTest.ts
│   │   ├── sheet
│   │   │   ├── index.ts
│   │   │   ├── parseContentSheet.ts
│   │   │   ├── route.ts
│   │   │   └── sheet.ts
│   │   ├── useResponsive.ts
│   │   └── write.ts
│   ├── types
│   │   ├── api
│   │   │   ├── holodex.ts
│   │   │   ├── mongoDB.ts
│   │   │   ├── schedule.ts
│   │   │   ├── setlist.ts
│   │   │   └── youtube.ts
│   │   └── time.ts
│   └── utils                                     # 유틸 함수 모음 폴더
│       ├── checkReferer.ts
│       ├── combineChannelData-v2.ts
│       ├── combineChannelData.ts
│       ├── convertTimeToSec.ts
│       ├── getCookie.ts
│       ├── getCssUrl.ts
│       ├── getPagenationRange.ts
│       ├── getTime.ts
│       ├── gtag.ts
│       ├── helper.ts
│       ├── parseAccessToken.ts
│       ├── parseMongoDBData.ts
│       ├── popup.ts
│       ├── regexp.ts
│       ├── renderSubscribe.ts
│       ├── windowEvent.ts
│       └── wrapTimeWithLink.ts
├── README.md
├── sentry.client.config.ts
├── sentry.edge.config.ts
├── sentry.server.config.ts
├── next-env.d.ts
├── next.config.ts
├── package.json
├── playwright-report
├── playwright.config.ts
├── pnpm-lock.yaml
├── biome.json
├── eslint.config.js
├── lefthook.yml
├── LICENSE
├── test-results
├── tsconfig.json
├── vercel.json
└── vitest.config.ts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yiminwook) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
