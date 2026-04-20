## launchsolo

> - React Router 7 as Framework Mode (Remix)


# Framework
- React Router 7 as Framework Mode (Remix)
- Created with "npx create-react-router@7.0.1 <my_app>"


## Code Structure
- route.ts: Route definition
- root.tsx: Route Module 을 렌더링할때 항상 먼저  root.tsx의 export default function 확인. Outlet: Placeholder 역할을 하는 component로 Route Module 이 렌더링되는 곳, Layout: App 을 감싸는 component
- ScrollRestoration: 페이지 이동후 이전페이지로 돌아갈때 스크롤 위치


### Data Fetching
1. By default, use loader function to fetch data. loader runs in backend, and run before UI is given to the user.
- Example:
```
export const meta: MetaFunction = () => {
  return [
    { title: "Home | wemake" },
    { name: "description", content: "Welcome to wemake" },
  ];
};

export const loader = async ({ request }: Route.LoaderArgs) => {
  const { client, headers } = makeSSRClient(request);
  const products = await getProductsByDateRange(client, {
    startDate: DateTime.now().startOf("day"),
    endDate: DateTime.now().endOf("day"),
    limit: 7,
  });

  const ideas = await getGptIdeas(client, { limit: 7 });

  return { products, ideas };
};

export default function HomePage({ loaderData }: Route.ComponentProps) {

```
- loader()는 UI 없이도 사용 가능
    - redirect() 로 다른페이지 연결
    - Response.json() 을 리턴하면 다른 앱의 backend API 로 사용 가능
2. .react-router: routes.ts 에 라우트가 추가될 때마다 자동으로 생성. .react-router/types/app 에 각 페이지별로 생성, 각 페이지의 props를 Route.ComponentProps 로 지정. 


### Links
각 페이지의  export const links 를 가져와서 head 영역의 link 에 추가되는 링크 리스트를 구성해 주는 컴포넌트

### Meta
각 페이지의 export const meta 를 가져와서 meta 태크 구성

## Libraries to use
luxon: date util
zod: user input validation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trinity31) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
