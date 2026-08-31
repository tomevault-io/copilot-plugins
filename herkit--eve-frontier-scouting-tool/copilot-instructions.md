## eve-frontier-scouting-tool

> A specialized tool for EVE Frontier scout route data that:

# Architecture

Server Side: node.js
Client Side: react

# What to build

A specialized tool for EVE Frontier scout route data that:
1. Takes two text inputs containing EVE Frontier system links from scout/mapping tools
2. Extracts all system links in the format `<a href="showinfo:5//XXXXXX">SystemName</a>`
3. Finds the intersection - systems that appear in BOTH inputs
4. Displays results with system names and IDs
5. Exports common systems back into the original format, batched by max 3900 characters per batch

## Key Features

- Extract EVE Frontier showinfo links from both inputs
- Match systems based on their unique system ID (not name)
- Display statistics: total links in each input, common links found
- Show all extracted systems with expandable details
- Export formatted batches with copy-to-clipboard functionality
- Each batch respects 3900 character limit for compatibility with other tools
- Links formatted as: `<a href="showinfo:5//ID">Name</a>→ <a href="showinfo:5//ID">Name</a>→ ...`

## Technical Details

### Link Format
- EVE Frontier uses `showinfo:5//XXXXXX` where XXXXXX is the system ID
- System IDs are unique identifiers (e.g., 30015856)
- System names are NOT unique (can have duplicates)
- Always match/compare by ID, not name

### Export Requirements
- Max 3900 characters per batch in the game client (with hidden markup)
- Game client adds exactly 82 characters of overhead per link (color codes, formatting, etc)
- Separators `→ ` do NOT get overhead - counted at face value (2 chars)
- Tool calculates precise in-game character count: `(linkCount × 82) + rawChars`
- Or more explicitly: `sum(linkLength + 82 for each link) + (separatorCount × 2)`
- Links joined with `→ ` (arrow + space = 2 chars)
- Format: `<a href="showinfo:5//ID">Name</a>→ <a href="showinfo:5//ID">Name</a>`
- Must maintain exact format for compatibility with scout tools
- UI displays both raw character count and expected in-game character count

### Code Organization
- Backend: `server/index.js` - All API endpoints and business logic
- Frontend: `client/src/App.js` - Main React component
- Styling: `client/src/App.css` - All UI styles

## Future Enhancements

Phase 2 will add intelligent route optimization:
- Extract map data from EVE Frontier (system positions, gate connections, distances)
- Build a graph database of the system network (nodes=systems, edges=gates)
- Implement pathfinding algorithms (Dijkstra's/A*) to calculate optimal routes
- Sort intersected systems by optimal path distance/order
- Display route information similar to input format with distances
- Generate optimized batches that follow efficient travel paths
- Show alternative routes and path comparisons

# Example inputs:

Input one:
```
Y:S638 → J:23O1
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015856">Y:S638</a> 0.00→ <a href="showinfo:5//30015856">Y:S638</a> (4)→ <a href="showinfo:5//30015857">Q:KKR4</a> 8.38→ <a href="showinfo:5//30015849">J:RS4O</a> 7.93→ <a href="showinfo:5//30015861">P:12RK</a> 9.73→ <a href="showinfo:5//30015848">M:13TN</a> 14.45→ <a href="showinfo:5//30015850">G:SE82</a> 27.49→ <a href="showinfo:5//30015902">P:1A91*</a> 15.91→ <a href="showinfo:5//30015904">Z:TIN6</a> 25.89→ <a href="showinfo:5//30015916">D:1408</a> 14.48→ <a href="showinfo:5//30015903">G:142V</a> 10.60→ <a href="showinfo:5//30015910">P:T7K3</a> 14.81→ <a href="showinfo:5//30014291">UB3-Q85</a> 9.67→ <a href="showinfo:5//30014296">ETC-J65*</a> 4.97→ <a href="showinfo:5//30014297">OMR-S75</a> 6.84→ <a href="showinfo:5//30015927">D:10L4</a> 6.65→ <a href="showinfo:5//30014290">O6R-KB5</a> 7.92→ <a href="showinfo:5//30015919">H:16KK</a> 14.96→ <a href="showinfo:5//30015931">Y:15S6</a> 7.21→ <a href="showinfo:5//30015921">Q:2E8R</a> 9.74→ <a href="showinfo:5//30015926">B:VA8E</a> 17.62→ <a href="showinfo:5//30015963">M:L47O</a> 13.14→ <a href="showinfo:5//30015957">J:SK3S</a> 5.00→ <a href="showinfo:5//30015972">M:10KO</a> 38.32→ <a href="showinfo:5//30007294">U:NEAT</a> 38.65→ <a href="showinfo:5//30015859">P:159E</a>

Y:S638 → J:23O1 (Page 2)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015859">P:159E</a> 37.76→ <a href="showinfo:5//30015858">B:1I60</a> 23.14→ <a href="showinfo:5//30015862">Q:19V1*</a> 11.77→ <a href="showinfo:5//30015853">F:O14O</a> 10.36→ <a href="showinfo:5//30012981">G.XFL.XXH</a> 17.37→ <a href="showinfo:5//30012983">C.3QL.S1B</a> 9.84→ <a href="showinfo:5//30012986">C.BPL.WSQ</a> 14.52→ <a href="showinfo:5//30012948">B.0BL.QEP</a> 22.91→ <a href="showinfo:5//30012924">M.5XL.N6M</a> 40.05→ <a href="showinfo:5//30012643">M.R2D.Y8L</a> 8.41→ <a href="showinfo:5//30012626">L.12D.2FR</a> 20.96→ <a href="showinfo:5//30012636">B.64D.18M</a> 7.15→ <a href="showinfo:5//30012980">B.F5D.B5M</a> 15.29→ <a href="showinfo:5//30012725">C.68D.G0V*</a> 8.94→ <a href="showinfo:5//30012726">M.S9D.EQV</a> 6.56→ <a href="showinfo:5//30012977">B.V9D.NJQ</a> 2.17→ <a href="showinfo:5//30012970">C.79D.3EQ</a> 5.43→ <a href="showinfo:5//30014301">IC8-B85*</a> 9.08→ <a href="showinfo:5//30012968">C.97D.68V</a> 15.45→ <a href="showinfo:5//30012967">G.Q5D.Q7P*</a> 10.23→ <a href="showinfo:5//30012966">M.K4D.M6C</a> 7.19→ <a href="showinfo:5//30012931">B.43D.DKD</a> 7.12→ <a href="showinfo:5//30012963">B.C3D.LGM</a> 4.57→ <a href="showinfo:5//30012976">B.24D.QTM</a> 12.63→ <a href="showinfo:5//30012936">M.31D.XJM</a>

Y:S638 → J:23O1 (Page 3)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012936">M.31D.XJM</a> 11.02→ <a href="showinfo:5//30015855">B:T3TI</a> 9.12→ <a href="showinfo:5//30015852">U:OO69</a> 24.76→ <a href="showinfo:5//30015846">M:K30N</a> 17.23→ <a href="showinfo:5//30015854">D:1S32</a> 21.06→ <a href="showinfo:5//30012947">B.9ZL.JNX</a> 17.58→ <a href="showinfo:5//30012946">B.GZL.DYF</a> 12.87→ <a href="showinfo:5//30012957">B.ZJL.M1Q*</a> 15.19→ <a href="showinfo:5//30012950">B.7ZL.K9V</a> 16.59→ <a href="showinfo:5//30015860">Y:163I</a> 16.95→ <a href="showinfo:5//30015847">B:SR54</a> 15.39→ <a href="showinfo:5//30012969">C.H2D.SDG</a> 4.99→ <a href="showinfo:5//30012971">C.C3D.KRB</a> 14.93→ <a href="showinfo:5//30012645">B.75D.EQF</a> 17.38→ <a href="showinfo:5//30012646">B.53D.L4Q</a> 2.91→ <a href="showinfo:5//30012644">B.S3D.9NQ*</a> 22.55→ <a href="showinfo:5//30012750">M.37D.M8Q</a> 8.21→ <a href="showinfo:5//30012746">B.06D.9ZP</a> 11.91→ <a href="showinfo:5//30012634">M.R6D.Z7P</a> 3.02→ <a href="showinfo:5//30012633">B.J6D.4BP</a> 4.45→ <a href="showinfo:5//30012635">M.Q6D.3YV*</a> 5.11→ <a href="showinfo:5//30012736">M.86D.KVQ</a> 6.74→ <a href="showinfo:5//30012751">B.G4D.HEV</a> 5.26→ <a href="showinfo:5//30012627">B.B4D.8FP</a> 7.30→ <a href="showinfo:5//30012647">M.E2D.5XM</a>

Y:S638 → J:23O1 (Page 4)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012647">M.E2D.5XM</a> 15.73→ <a href="showinfo:5//30012631">B.PEL.2JV</a> 3.10→ <a href="showinfo:5//30012630">B.JKL.HEV</a> 26.24→ <a href="showinfo:5//30013018">H.GJL.RSD*</a> 7.38→ <a href="showinfo:5//30015947">G:13VK</a> 6.99→ <a href="showinfo:5//30015952">U:1IOE</a> 12.98→ <a href="showinfo:5//30015936">M:1599</a> 5.58→ <a href="showinfo:5//30015949">M:134O</a> 11.50→ <a href="showinfo:5//30015951">Z:162V</a> 6.97→ <a href="showinfo:5//30015942">M:1IRS</a> 6.38→ <a href="showinfo:5//30015940">G:TTK8</a> 15.98→ <a href="showinfo:5//30015941">M:ERL4</a> 16.91→ <a href="showinfo:5//30015954">M:R98V</a> (1)→ <a href="showinfo:5//30015950">G:TTI8</a> 13.69→ <a href="showinfo:5//30015953">Q:K3S4</a> 18.52→ <a href="showinfo:5//30012915">M.BQL.T9P</a> 10.10→ <a href="showinfo:5//30012911">B.5GL.W1P</a> 9.64→ <a href="showinfo:5//30012919">M.DGL.W1Q*</a> 11.60→ <a href="showinfo:5//30012918">B.9FL.K5Q</a> 8.19→ <a href="showinfo:5//30012914">M.7QL.6YP</a> 8.54→ <a href="showinfo:5//30012910">M.RBL.VEP</a> 11.14→ <a href="showinfo:5//30012913">M.XHL.6QQ</a> 8.47→ <a href="showinfo:5//30012916">C.HJL.GWV</a> 11.41→ <a href="showinfo:5//30015851">Z:1344</a> 11.41→ <a href="showinfo:5//30015937">P:140V</a> 9.87→ <a href="showinfo:5//30015943">Y:12E8*</a>

Y:S638 → J:23O1 (Page 5)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015943">Y:12E8*</a> 3.91→ <a href="showinfo:5//30015935">D:RN1A</a> 9.55→ <a href="showinfo:5//30015939">P:2NR8</a> 20.72→ <a href="showinfo:5//30015048">F:13N9</a> 18.30→ <a href="showinfo:5//30012756">M.N2D.W0G</a> 7.51→ <a href="showinfo:5//30012752">B.K1D.BZH</a> 8.98→ <a href="showinfo:5//30012741">B.S0D.TDG</a> 4.59→ <a href="showinfo:5//30012738">M.N0D.19B</a> 2.66→ <a href="showinfo:5//30012748">M.21D.P6B</a> 2.71→ <a href="showinfo:5//30012747">M.E0D.ZNB</a> 6.49→ <a href="showinfo:5//30012739">B.G0D.J9B</a> 13.85→ <a href="showinfo:5//30012744">M.54D.L9G</a> 2.57→ <a href="showinfo:5//30012745">M.H4D.KEB</a> 27.23→ <a href="showinfo:5//30015885">M:1285</a> 14.96→ <a href="showinfo:5//30012743">B.P7D.6SJ</a> 10.70→ <a href="showinfo:5//30015929">Q:1IA5*</a> 6.08→ <a href="showinfo:5//30012735">B.16D.PGG</a> 7.32→ <a href="showinfo:5//30012755">B.J4D.1RB</a> 6.26→ <a href="showinfo:5//30012753">B.64D.5RF</a> 7.15→ <a href="showinfo:5//30012754">B.T2D.C2F</a> 12.01→ <a href="showinfo:5//30000003">U 3183</a> 11.24→ <a href="showinfo:5//30012620">M.C2D.27C</a> 7.99→ <a href="showinfo:5//30012628">M.M2D.RKM</a> 4.04→ <a href="showinfo:5//30012638">B.Q2D.4KP</a> 3.38→ <a href="showinfo:5//30012740">B.E2D.0BV</a>

Y:S638 → J:23O1 (Page 6)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012740">B.E2D.0BV</a> 5.44→ <a href="showinfo:5//30012742">B.73D.EGV</a> 3.64→ <a href="showinfo:5//30012641">C.S2D.SRV</a> 5.52→ <a href="showinfo:5//30012632">B.W0D.98V</a> 13.25→ <a href="showinfo:5//30012637">B.4EL.MFM</a> 6.02→ <a href="showinfo:5//30012640">C.7KL.0NP</a> 10.04→ <a href="showinfo:5//30012927">B.0KL.59P</a> 10.99→ <a href="showinfo:5//30012921">B.EZL.8BD</a> 12.18→ <a href="showinfo:5//30012926">M.LEL.Z7L</a> 7.28→ <a href="showinfo:5//30012925">B.DKL.J0D</a> 8.42→ <a href="showinfo:5//30012956">B.FWL.FJC</a> 10.01→ <a href="showinfo:5//30012955">B.JKL.13V</a> 7.43→ <a href="showinfo:5//30012962">B.F0D.EMV</a> 12.36→ <a href="showinfo:5//30012965">B.C2D.10F</a> 15.40→ <a href="showinfo:5//30012949">B.B0D.E1B</a> 12.66→ <a href="showinfo:5//30015911">D:16LA</a> 4.31→ <a href="showinfo:5//30015908">Q:19TE</a> 8.27→ <a href="showinfo:5//30014300">OTQ-675</a> (4)→ <a href="showinfo:5//30014287">A42-NC5</a> (3)→ <a href="showinfo:5//30014298">O01-Q65</a> 9.84→ <a href="showinfo:5//30012964">M.75D.MDQ</a> 2.48→ <a href="showinfo:5//30012978">B.V5D.T0F</a> 7.26→ <a href="showinfo:5//30012974">B.P5D.FGB</a> 6.63→ <a href="showinfo:5//30012975">M.46D.9TH</a> 7.41→ <a href="showinfo:5//30012979">B.H6D.YGJ</a>

Y:S638 → J:23O1 (Page 7)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012979">B.H6D.YGJ</a> 5.48→ <a href="showinfo:5//30012973">M.Z5D.CZX</a> 13.50→ <a href="showinfo:5//30015962">B:N016</a> 23.28→ <a href="showinfo:5//30015923">J:23O1*</a>
``` 

Input two:
```
O01-Q65 → M.5XL.N6M
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30014298">O01-Q65</a> 0.00→ <a href="showinfo:5//30014298">O01-Q65</a> (1)→ <a href="showinfo:5//30014294">AP9-475</a> 8.43→ <a href="showinfo:5//30015916">D:1408</a> 11.31→ <a href="showinfo:5//30015910">P:T7K3</a> 10.60→ <a href="showinfo:5//30015903">G:142V</a> 7.99→ <a href="showinfo:5//30015907">U:1062</a> 30.67→ <a href="showinfo:5//30015914">H:109R</a> 17.87→ <a href="showinfo:5//30015904">Z:TIN6</a> 15.91→ <a href="showinfo:5//30015902">P:1A91*</a> 33.17→ <a href="showinfo:5//30015906">G:10IT</a> 26.98→ <a href="showinfo:5//30015957">J:SK3S</a> 13.14→ <a href="showinfo:5//30015963">M:L47O</a> 33.03→ <a href="showinfo:5//30015918">F:141N</a> 14.74→ <a href="showinfo:5//30015923">J:23O1*</a> 11.47→ <a href="showinfo:5//30015926">B:VA8E</a> 12.24→ <a href="showinfo:5//30014291">UB3-Q85</a> 10.98→ <a href="showinfo:5//30012979">B.H6D.YGJ</a> 5.48→ <a href="showinfo:5//30012973">M.Z5D.CZX</a> 13.50→ <a href="showinfo:5//30015962">B:N016</a> 12.62→ <a href="showinfo:5//30015850">G:SE82</a> 14.45→ <a href="showinfo:5//30015848">M:13TN</a> 7.37→ <a href="showinfo:5//30015856">Y:S638</a> (3)→ <a href="showinfo:5//30015857">Q:KKR4</a> 8.38→ <a href="showinfo:5//30015849">J:RS4O</a> 7.93→ <a href="showinfo:5//30015861">P:12RK</a>

O01-Q65 → M.5XL.N6M (Page 2)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015861">P:12RK</a> 24.23→ <a href="showinfo:5//30015860">Y:163I</a> 9.20→ <a href="showinfo:5//30015846">M:K30N</a> 12.22→ <a href="showinfo:5//30012947">B.9ZL.JNX</a> 18.62→ <a href="showinfo:5//30012952">M.QXL.RRG</a> 10.28→ <a href="showinfo:5//30012961">B.1WL.45H</a> 15.20→ <a href="showinfo:5//30012946">B.GZL.DYF</a> 12.87→ <a href="showinfo:5//30012957">B.ZJL.M1Q*</a> 15.19→ <a href="showinfo:5//30012950">B.7ZL.K9V</a> 23.14→ <a href="showinfo:5//30012954">M.WKL.N3D</a> 27.01→ <a href="showinfo:5//30012938">B.Q4D.QWR</a> 24.22→ <a href="showinfo:5//30012937">B.R0D.9MM</a> 35.00→ <a href="showinfo:5//30012943">B.37D.67G</a> 22.59→ <a href="showinfo:5//30015909">D:2NEI</a> 19.35→ <a href="showinfo:5//30014286">E1K-RG5</a> 5.70→ <a href="showinfo:5//30015913">D:TOI6</a> 28.76→ <a href="showinfo:5//30014295">ATS-GP5</a> 24.88→ <a href="showinfo:5//30014800">C.JMD.0VP*</a> 25.54→ <a href="showinfo:5//30014805">B.MLD.XYN</a> 34.98→ <a href="showinfo:5//30014798">M.NMD.YKL</a> 51.39→ <a href="showinfo:5//30014293">I4V-NB5</a> (6)→ <a href="showinfo:5//30014283">ISJ-4F5</a> (5)→ <a href="showinfo:5//30014300">OTQ-675</a> 8.27→ <a href="showinfo:5//30015908">Q:19TE</a> 4.31→ <a href="showinfo:5//30015911">D:16LA</a>

O01-Q65 → M.5XL.N6M (Page 3)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015911">D:16LA</a> 12.66→ <a href="showinfo:5//30012949">B.B0D.E1B</a> 15.40→ <a href="showinfo:5//30012965">B.C2D.10F</a> 12.36→ <a href="showinfo:5//30012962">B.F0D.EMV</a> 7.43→ <a href="showinfo:5//30012955">B.JKL.13V</a> 10.01→ <a href="showinfo:5//30012956">B.FWL.FJC</a> 8.42→ <a href="showinfo:5//30012925">B.DKL.J0D</a> 7.28→ <a href="showinfo:5//30012926">M.LEL.Z7L</a> 12.18→ <a href="showinfo:5//30012921">B.EZL.8BD</a> 10.99→ <a href="showinfo:5//30012927">B.0KL.59P</a> 10.04→ <a href="showinfo:5//30012640">C.7KL.0NP</a> 6.02→ <a href="showinfo:5//30012637">B.4EL.MFM</a> 13.25→ <a href="showinfo:5//30012632">B.W0D.98V</a> 5.52→ <a href="showinfo:5//30012641">C.S2D.SRV</a> 3.64→ <a href="showinfo:5//30012742">B.73D.EGV</a> 5.44→ <a href="showinfo:5//30012740">B.E2D.0BV</a> 8.17→ <a href="showinfo:5//30012753">B.64D.5RF</a> 6.26→ <a href="showinfo:5//30012755">B.J4D.1RB</a> 7.32→ <a href="showinfo:5//30012735">B.16D.PGG</a> 6.08→ <a href="showinfo:5//30015929">Q:1IA5*</a> 10.70→ <a href="showinfo:5//30012743">B.P7D.6SJ</a> 19.14→ <a href="showinfo:5//30015931">Y:15S6</a> 7.21→ <a href="showinfo:5//30015921">Q:2E8R</a> 9.41→ <a href="showinfo:5//30015919">H:16KK</a> 7.18→ <a href="showinfo:5//30015934">H:16AO</a>

O01-Q65 → M.5XL.N6M (Page 4)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30015934">H:16AO</a> 8.30→ <a href="showinfo:5//30015933">G:R0A1</a> 11.19→ <a href="showinfo:5//30014290">O6R-KB5</a> 6.65→ <a href="showinfo:5//30015927">D:10L4</a> 6.84→ <a href="showinfo:5//30014297">OMR-S75</a> 4.97→ <a href="showinfo:5//30014296">ETC-J65*</a> 7.20→ <a href="showinfo:5//30012975">M.46D.9TH</a> 6.63→ <a href="showinfo:5//30012974">B.P5D.FGB</a> 7.26→ <a href="showinfo:5//30012978">B.V5D.T0F</a> 2.48→ <a href="showinfo:5//30012964">M.75D.MDQ</a> 10.21→ <a href="showinfo:5//30012971">C.C3D.KRB</a> 4.99→ <a href="showinfo:5//30012969">C.H2D.SDG</a> 15.39→ <a href="showinfo:5//30015847">B:SR54</a> 15.79→ <a href="showinfo:5//30015852">U:OO69</a> 9.12→ <a href="showinfo:5//30015855">B:T3TI</a> 11.02→ <a href="showinfo:5//30012936">M.31D.XJM</a> 12.63→ <a href="showinfo:5//30012976">B.24D.QTM</a> 4.57→ <a href="showinfo:5//30012963">B.C3D.LGM</a> 7.12→ <a href="showinfo:5//30012931">B.43D.DKD</a> 7.19→ <a href="showinfo:5//30012966">M.K4D.M6C</a> 6.10→ <a href="showinfo:5//30012944">M.66D.S6D</a> 6.52→ <a href="showinfo:5//30012734">B.D5D.8CL</a> 6.91→ <a href="showinfo:5//30012721">B.96D.PMR</a> 11.50→ <a href="showinfo:5//30012719">B.H7D.5KD</a> 10.41→ <a href="showinfo:5//30012972">C.59D.E0P</a>

O01-Q65 → M.5XL.N6M (Page 5)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012972">C.59D.E0P</a> 9.21→ <a href="showinfo:5//30012718">B.7SD.PXC</a> 5.31→ <a href="showinfo:5//30012731">B.DTD.47C</a> 6.50→ <a href="showinfo:5//30012730">B.YTD.YML</a> 9.20→ <a href="showinfo:5//30012732">M.JND.FLD</a> 2.11→ <a href="showinfo:5//30012722">B.TRD.YSD</a> 16.42→ <a href="showinfo:5//30012729">B.99D.BHL</a> 7.42→ <a href="showinfo:5//30012733">M.58D.NKR</a> 14.03→ <a href="showinfo:5//30012724">O.99D.7KC</a> 9.16→ <a href="showinfo:5//30012725">C.68D.G0V*</a> 8.94→ <a href="showinfo:5//30012726">M.S9D.EQV</a> 6.22→ <a href="showinfo:5//30012728">B.VSD.56V</a> 7.31→ <a href="showinfo:5//30012977">B.V9D.NJQ</a> 2.17→ <a href="showinfo:5//30012970">C.79D.3EQ</a> 5.43→ <a href="showinfo:5//30014301">IC8-B85*</a> 9.08→ <a href="showinfo:5//30012968">C.97D.68V</a> 11.70→ <a href="showinfo:5//30012980">B.F5D.B5M</a> 7.15→ <a href="showinfo:5//30012636">B.64D.18M</a> 17.26→ <a href="showinfo:5//30012645">B.75D.EQF</a> 17.38→ <a href="showinfo:5//30012646">B.53D.L4Q</a> 2.91→ <a href="showinfo:5//30012644">B.S3D.9NQ*</a> 7.77→ <a href="showinfo:5//30012751">B.G4D.HEV</a> 5.26→ <a href="showinfo:5//30012627">B.B4D.8FP</a> 7.30→ <a href="showinfo:5//30012647">M.E2D.5XM</a> 13.71→ <a href="showinfo:5//30012634">M.R6D.Z7P</a>

O01-Q65 → M.5XL.N6M (Page 6)
Gate: (x)→ SmartGate: []→ Jump: <distance>→ | * = potential heat trap
<a href="showinfo:5//30012634">M.R6D.Z7P</a> 3.02→ <a href="showinfo:5//30012633">B.J6D.4BP</a> 4.45→ <a href="showinfo:5//30012635">M.Q6D.3YV*</a> 5.11→ <a href="showinfo:5//30012736">M.86D.KVQ</a> 17.45→ <a href="showinfo:5//30012727">M.B9D.1VM</a> 19.61→ <a href="showinfo:5//30015925">Y:150A</a> 12.64→ <a href="showinfo:5//30015922">D:168V</a> 6.04→ <a href="showinfo:5//30015932">Q:1653</a> 12.56→ <a href="showinfo:5//30015928">P:S21R</a> 5.17→ <a href="showinfo:5//30015924">H:18EV</a> 16.65→ <a href="showinfo:5//30014285">E0C-TD5</a> 3.93→ <a href="showinfo:5//30014299">A00-8D5</a> 16.35→ <a href="showinfo:5//30014812">B.TTD.VMC</a> 20.53→ <a href="showinfo:5//30012723">C.S9D.Z1R</a> 5.13→ <a href="showinfo:5//30012720">B.SSD.P5N</a> 29.11→ <a href="showinfo:5//30012935">C.W5D.CVC</a> 7.19→ <a href="showinfo:5//30012967">G.Q5D.Q7P*</a> 43.67→ <a href="showinfo:5//30012626">L.12D.2FR</a> 13.71→ <a href="showinfo:5//30012920">B.JEL.6XN</a> 27.48→ <a href="showinfo:5//30012924">M.5XL.N6M</a>
```

---
> Source: [herkit/eve-frontier-scouting-tool](https://github.com/herkit/eve-frontier-scouting-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
