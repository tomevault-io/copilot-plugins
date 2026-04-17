## events

> This repository is a personal assistant for finding interesting events and activities for our family.

# Family Events Assistant

This repository is a personal assistant for finding interesting events and activities for our family.

## Family Profile

- **Family size**: 3
- **Members**: Husband, Wife, Son (6 years old, born ~2020)
- **Location**: Zaandam, Netherlands
- **Languages**: English preferred (Dutch is limited)
- **Travel radius**: Flexible - Zaandam, Amsterdam, and beyond. Location closeness is nice but not a hard requirement.

## Preferences

### What We Want
1. **Creative/building activities**: workshops where we build, make, or create something together
2. **Brain challenges**: escape rooms, puzzles, scavenger hunts, quiz nights - things that use intelligence
3. **Competitive activities**: events where we compete as a family team against other families
4. **Social/interactive**: activities where we meet and interact with other families, not just our own
5. **Physical challenges**: climbing, obstacle courses, sports - active things we do together

### What We DON'T Want
- Generic tourist attractions or sightseeing (we're locals, not tourists)
- Generic national holidays like King's Day (obvious, not really for kids)
- Passive activities (just watching something)
- Drop-off activities (must be together as family)
- "Just another museum visit" without interactive/creative element

### Key Criteria
- **Goal**: Spend quality time together as a family
- **Age-appropriate**: Must be suitable for a 6-year-old (update age as child grows)
- **International-friendly**: Prefer English-friendly activities, but not a hard requirement
- **Together**: Activities the whole family can enjoy together (not drop-off activities)
- **Engaging**: Use our intelligence, creativity, or physical skills - not passive entertainment
- **Social**: Prefer activities where we interact with other families when possible

### Budget
- Paid events are totally fine - don't focus on free events
- Open to all price ranges
- Note costs when researching so family can decide

## How to Use This Repository

### Quick Event Search
Ask Claude to search for events:
- "Find events for this weekend"
- "What's happening in spring break?"
- "Find outdoor activities for next month"
- "Search for new family activities"

### Run the Search Helper Script
```bash
./scripts/search_events.sh [category] [timeframe]
# Categories: all, physical, brain, outdoor, seasonal, deals
# Timeframe: weekend, week, month, spring, summer
# Add --open as 3rd arg to open URLs in browser
```

### Repository Structure
- `CLAUDE.md` - This file (family profile, preferences, quick reference)
- `events/creative-competitive.md` - **BEST LIST**: Creative, competitive, social activities (build, compete, solve puzzles)
- `family-activities-zaandam-area.md` - Comprehensive activity guide with 30+ venues, costs, details
- `scripts/sources.md` - 50+ curated event sources organized by type (English/Dutch/local/booking)
- `scripts/search_events.sh` - Search helper script with prompts and links
- `events/visited.md` - Events we've been to (with ratings)
- `events/wishlist.md` - Events we want to attend
- `events/seasonal.md` - Recurring seasonal events calendar

## Event Sources (Quick Reference)

See `scripts/sources.md` for the full list (50+ sources). Top sources by use case:

### English-Friendly
- [I amsterdam Family](https://www.iamsterdam.com/en/see-and-do/family-and-kids) - Best English event calendar
- [Amsterdam Mamas](https://amsterdam-mamas.nl/) - Expat family community, 22,000+ members
- [TimeOut Amsterdam Kids](https://www.timeout.com/amsterdam/kids) - Curated English recommendations
- [AmsterdamKids.com](https://amsterdamkids.com/) - Family offers and event early access
- [IamExpat.nl](https://www.iamexpat.nl/lifestyle/events-festivals-netherlands) - Expat events (has RSS)

### Local Zaandam
- [Go-Kids Zaanland](https://go-kids.nl/zaanland/agenda) - Weekly local family event agenda
- [De Orkaan Waarheen](https://waarheen.deorkaan.nl/agenda/) - 274+ local events
- [Rodi.nl Zaanstad](https://www.rodi.nl/zaanstad/uit/) - Weekly weekend tips (every Friday)
- [Zaanse Schans Events](https://www.dezaanseschans.nl/en/zaanse-schans-events/) - Local events (free for residents)

### Dutch Family Sites (use translator)
- [Uitmetkinderen.nl](https://www.uitmetkinderen.nl) - 3,000+ family outings, has app
- [Kidsproof Amsterdam](https://www.kidsproof.nl/amsterdam) - Kid-tested activities
- [DagjeWeg.nl](https://www.dagjeweg.nl/) - 5,000+ day trips

### Booking & Deals
- [Tiqets](https://www.tiqets.com/en/amsterdam) - Museum/attraction tickets with deals
- [Groupon NL](https://www.groupon.nl) - Discounted family activities

## Top Activities Quick Reference

### Creative, Competitive & Social (Our Favorites!)
See `events/creative-competitive.md` for full details. Highlights:

| Activity | Type | Age | Cost/person | Why |
|----------|------|-----|-------------|-----|
| parkrun Amsterdamse Bos | Community run | 4+ | FREE | Meet families weekly, build social network |
| BAZ Bouldergym (Zaandam!) | Bouldering | 5+ | 10 EUR | Right here, brain + physical |
| Laser Tag (UP Events) | Team battles | 6+ | From 10.99 EUR | Your son CAN do this now |
| Sherlocked Escape Room | Brain puzzles | 6+ | 30-40 EUR | Best escape room for age 6 |
| Mystery.city Treasure Hunt | Physical puzzle hunt | 6+ | 25/15 EUR | Real locked boxes, not an app |
| Hockey Tournament (May 3!) | Team sport vs families | All | TBD | Exactly what we want |
| Strong Viking Family Run (Sep!) | Obstacle course | 5+ | Check site | Compete vs families, get muddy |
| Hemlab Science Lab (Zaandam!) | Experiments | 6+ | 14.20 EUR | Right in Zaandam |
| Buurman Woodworking | Build furniture | Ask | 95 EUR half-day | Build real things together |
| NEMO Maker Space | Invent/build | 4+ | 17.50 EUR | Hands-on making |

### Communities to Join
| Group | What | Link |
|-------|------|------|
| MOOM Amsterdam (409 members) | Family meetups, brunches, events | meetup.com/moomamsterdam |
| Amsterdam Intl Mamas (571 members) | Playdates, social for internationals | meetup.com/amsterdam-international-mamas-meetup |
| Coffee+Kids | Creative workshops for expat families | coffeeandkids.nl |

### In/Near Zaandam (0-10 min)
| Venue | What | Cost (per person) | English |
|-------|------|-------------------|---------|
| BAZ Bouldergym | Indoor bouldering, age 5+ | ~10 EUR | Yes |
| Hemlab Science Lab | Chemistry/physics experiments | ~14-23 EUR | Dutch (visual/hands-on) |
| Atelier Ainoa | Ceramics workshop | Contact | Yes |
| Verkade Experience | Cookie factory building | ~15-26.55 EUR | Yes |
| Zaanse Schans | Windmills, open-air walks, chocolate | Free to walk (17.50 museum) | Yes |
| Street Jump Zaanstad | Trampoline park (Wormerveer) | ~12-18 EUR | Limited |

### Amsterdam (15-30 min)
| Venue | What | Cost (per person) | English |
|-------|------|-------------------|---------|
| Laser Tag (UP Events) | Team battles, kids arena age 6+ | From 10.99 EUR | Yes |
| Sherlocked Escape Room | 90-min magical escape room, age 6+ | 30-40 EUR | Yes |
| NEMO Science Museum | 5 floors hands-on science + Maker Space | 17.50 EUR (4+) | Yes |
| Mystery.city | Physical treasure hunt with locked boxes | 25/15 EUR | Yes |
| parkrun Amsterdamse Bos | FREE weekly community run (Sat 9AM) | FREE | Yes |
| Fun Forest | Tree-top climbing, age 6+/110cm | 11.50-21.50 EUR | Yes |
| Mooie Boules | Jeu de boules + food hall | 10 EUR/45min | Yes |
| Aloha Amsterdam | Bowling + laser tag + mini golf | 37.50 EUR/lane/hr | Yes |
| EYE Film Museum | Make your own film | ~7-12 EUR | Yes |

### Day Trips (30-90 min)
| Venue | What | Cost (per person) | English |
|-------|------|-------------------|---------|
| Linnaeushof | Europe's largest playground, 350+ structures | 15-18 EUR | Yes |
| Keukenhof | Tulip gardens (Mar 19-May 10) | From 21.50 EUR | Yes |
| Efteling | Fairy-tale theme park | 40-50 EUR | Yes |
| Zuiderzee Museum | Open-air historic village (Enkhuizen) | ~10-18 EUR | Yes |

## Upcoming Seasonal Events (2026)

| Event | When | Notes |
|-------|------|-------|
| Keukenhof | Mar 19 - May 10 | Tulips! Book online, go weekday if possible |
| Koningsdag (King's Day) | April 27 | FREE, orange everything, flea markets |
| Slag om de Zaan | April 18 | Rowing on the Zaan, free to watch |
| Pinksterzaan | Whit Monday (June) | Zaanse Schans crafts & workshops |
| Vondelpark Theatre | June-September | Free shows, Sunday kids shows 14:00 |
| Zaandam Kermis | Sep 4-13 | Annual funfair |
| Sinterklaas Intocht | Mid-November | Dutch children's tradition |
| Amsterdam Light Festival | Nov-Jan | Light art on canals |

## Suggested Weekly Check Routine
1. **Friday**: Check Rodi.nl Zaanstad weekend tips + Kidsproof Amsterdam
2. **Weekly**: Scan Amsterdam Mamas newsletter + IamExpat events
3. **Monthly**: Review Go-Kids Zaanland agenda + DagjeWeg calendar
4. **Seasonally**: Check events/seasonal.md + Kermisplanner for fairs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faridmovsumov) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
