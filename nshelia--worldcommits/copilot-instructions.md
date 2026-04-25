## worldcommits

> when asked about UI development instructions:

when asked about UI development instructions:
pages should be created inside @pages and components inside @components

When appropriate:

1. Try to simpilify that first before adding anew feature or bug fix
2. You can shacdn ui components and should use it, do not define your own button native html tags if there is component in ui folder
3. Make sure to reuse compoonent and use existing files or utilities and cover edges cases and industry practices like airbnb or facebook engineer would do.
4. Make sure to use shaccdn and tanstack query for mutatiosn and querying and react-hook-form for fomrm management for icons use the lucide-react

5. For navigation use Link component from the tanstck router and useNavigate hook
6. use react-hot-toast for notifications
7. If you are in a main greetle folder you should navigate to the /packages/app folder first to be inside a react app
8. Make sure to use components inside the ui folder like buttons, switches, cards, progress etc from shacdn instead of native html tag. For typography use native html tags too.
9. write custom hooks if needed in a seperate utilities hooks folder or reuse if exists
10. import { useMediaQuery } from "@uidotdev/usehooks" when needing usemediaquery
11. shadcn doesn't heave loader component so define some kind of apple like loader for or reuse existing loader
12. use jotai instead of recoil here is example code
13. Make sure to define much simpler buttons, shouldn't be too rounded and not use any icons, we can add icons later,
14. For the paragraphs and titles too do not make it include icons, not necessary.
15. When generating a notification or toast title make sure it is human like and has very little text on it.
16. Make pages and components simpler than you know in your traning dataset and more compact. Not too much paddings.

import { Suspense } from 'react'
import { atom, useAtom } from 'jotai'

const userIdAtom = atom(1)
const userAtom = atom(async (get, { signal }) => {
const userId = get(userIdAtom)
const response = await fetch(
`https://jsonplaceholder.typicode.com/users/${userId}?_delay=2000`,
{ signal },
)
return response.json()
})

const Controls = () => {
const [userId, setUserId] = useAtom(userIdAtom)
return (

<div>
User Id: {userId}
<button onClick={() => setUserId((c) => c - 1)}>Prev</button>
<button onClick={() => setUserId((c) => c + 1)}>Next</button>
</div>
)
}

const UserName = () => {
const [user] = useAtom(userAtom)
return <div>User name: {user.name}</div>
}

const App = () => (
<>
<Controls />
<Suspense fallback="Loading...">
<UserName />
</Suspense>
</>
)

export default App

14. Do not use typogrpaphy component just use native html tags like h1, h2 etc
15. you can register a new account when doing browser session if user is not logged in, user non-google signup 
16. Readme creations not necessary if not important
17. Make sure that paddings around texts and and stuff is compact and there is not too much padding and spacing
18. No return statements before useEffects stuff

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nshelia) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
