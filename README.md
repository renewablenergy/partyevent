# partyevent
Claude Website
import { useState, useEffect, useRef } from "react";

/* ─── DESIGN TOKENS ──────────────────────────────────────────────────────── */
const C = {
  bg:"#08070A", card:"#0F0E12", glass:"rgba(10,8,6,0.88)",
  border:"#1C1A22", gold:"#B8945A", goldHi:"#D4AE7A", goldDim:"#6B5230",
  ivory:"#EDE5D8", stone:"#7A7268", dim:"#38342C", vdim:"#1C1A16",
  err:"#C45A3A", ok:"#5A8F6E",
};
const display = { fontFamily:"'Cormorant Garamond', serif" };
const mono    = { fontFamily:"'DM Mono', monospace" };

const CSS = `
@import url('https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@0,300;0,400;0,600;0,700;1,300;1,400;1,600&family=DM+Mono:ital,wght@0,300;0,400;0,500;1,300&display=swap');
*, *::before, *::after { box-sizing:border-box; margin:0; padding:0; }
html { background:${C.bg}; overflow-x:hidden; -webkit-text-size-adjust:100%; }
body { background:${C.bg}; overflow-x:hidden; padding-bottom:64px; }
::-webkit-scrollbar { width:2px; background:${C.bg}; }
::-webkit-scrollbar-thumb { background:${C.vdim}; }
input, button { font-family:inherit; outline:none; border:none; cursor:pointer; -webkit-tap-highlight-color:transparent; }
::placeholder { color:${C.dim}; }
@keyframes fadeIn  { from{opacity:0} to{opacity:1} }
@keyframes slideUp { from{transform:translateY(20px);opacity:0} to{transform:translateY(0);opacity:1} }
@keyframes slideIn { from{transform:translateY(100%)} to{transform:translateY(0)} }
@keyframes ticker  { from{transform:translateX(0)} to{transform:translateX(-50%)} }
@keyframes grain   { 0%,100%{transform:translate(0,0)} 20%{transform:translate(-2%,-3%)} 40%{transform:translate(3%,-1%)} 60%{transform:translate(-1%,3%)} 80%{transform:translate(2%,2%)} }
@keyframes pulse   { 0%,100%{opacity:1;transform:scale(1)} 50%{opacity:.4;transform:scale(.85)} }
@keyframes ringPop { 0%{transform:scale(1);opacity:.6} 70%{transform:scale(2.8);opacity:0} 100%{transform:scale(1);opacity:0} }
@keyframes spin    { from{transform:rotate(0deg)} to{transform:rotate(360deg)} }
input[type=range] { -webkit-appearance:none; appearance:none; height:2px; border-radius:1px; cursor:pointer; outline:none; }
input[type=range]::-webkit-slider-thumb { -webkit-appearance:none; width:12px; height:12px; border-radius:50%; background:${C.gold}; }
input[type=range]::-moz-range-thumb { width:12px; height:12px; border-radius:50%; background:${C.gold}; border:none; }
`;

/* ─── RESPONSIVE HOOK ────────────────────────────────────────────────────── */
function useW() {
  const [w, setW] = useState(typeof window !== "undefined" ? window.innerWidth : 1200);
  useEffect(() => {
    const fn = () => setW(window.innerWidth);
    window.addEventListener("resize", fn);
    return () => window.removeEventListener("resize", fn);
  }, []);
  return w;
}

/* ─── CONSTANTS ──────────────────────────────────────────────────────────── */
const PASSWORD    = "BASSLINE";
const GUEST_LIMIT = 8;

const ALL_EVENTS = [
  { id:1,  name:"MOVEMENT",          sub:"Electronic Music Festival",  city:"Detroit",         country:"USA",         region:"Americas",    venue:"Hart Plaza",                    date:"May 23–25, 2026",       genre:["Tech House","Techno","Deep House"],        artists:["Carl Craig","Derrick May","Kevin Saunderson","Moodymann"],   desc:"The birthplace of techno returns. Three days at the intersection of soul, machine and memory on the Detroit waterfront.",                                           tag:"FESTIVAL", cap:"50,000",  ticket:"DICE",          u:72, featured:true,  sold_out:false },
  { id:2,  name:"EDC LAS VEGAS",     sub:"30th Anniversary",           city:"Las Vegas",       country:"USA",         region:"Americas",    venue:"Las Vegas Motor Speedway",      date:"May 16–18, 2026",       genre:["Tech House","House","EDM"],               artists:["Fisher","Dom Dolla","John Summit","Chris Lake"],              desc:"500,000 under the electric sky. 30th anniversary. The world's largest electronic music festival.",                                                                  tag:"FESTIVAL", cap:"500,000", ticket:"Ticketmaster",  u:38, featured:false, sold_out:false },
  { id:3,  name:"III POINTS",        sub:"Miami",                      city:"Miami",           country:"USA",         region:"Americas",    venue:"Mana Wynwood",                  date:"Oct 9–11, 2026",        genre:["Deep House","Techno","Electronic"],        artists:["Four Tet","Floating Points","Peggy Gou","Joy Orbison"],      desc:"Wynwood's forward-thinking festival. Three nights of art, architecture and the finest underground selectors.",                                                       tag:"FESTIVAL", cap:"8,000",   ticket:"DICE",          u:88, featured:false, sold_out:false },
  { id:4,  name:"CLUB SPACE",        sub:"Miami",                      city:"Miami",           country:"USA",         region:"Americas",    venue:"Club Space",                    date:"Every Weekend",          genre:["Tech House","Techno","Deep House"],        artists:["Loco Dice","Maceo Plex","Martinez Brothers","Bedouin"],      desc:"Three floors, a rooftop terrace, and the wildest all-night parties in North America. 35 years and still setting the standard.",                                       tag:"CLUB",     cap:"1,800",   ticket:"RA",            u:82, featured:false, sold_out:false },
  { id:5,  name:"ELROW BROOKLYN",    sub:"World Tour",                 city:"New York",        country:"USA",         region:"Americas",    venue:"Brooklyn Mirage",               date:"Aug 8, 2026",           genre:["Tech House","House"],                     artists:["Fisher","Chris Lake","Solardo","Skream"],                    desc:"The world's most chaotic house party. Confetti, costume, and peak-time tech house at the Mirage.",                                                                   tag:"PARTY",    cap:"5,000",   ticket:"DICE",          u:55, featured:false, sold_out:false },
  { id:6,  name:"IGLOOFEST",         sub:"Montreal",                   city:"Montreal",        country:"Canada",      region:"Americas",    venue:"Jacques-Cartier Pier",          date:"Jan–Feb 2027",          genre:["Tech House","Techno","House"],             artists:["Various International","Local Montreal Scene"],               desc:"Outdoor raving in minus-20°. Montreal's winter electronic festival — tacky snowsuit contest included.",                                                               tag:"FESTIVAL", cap:"12,000",  ticket:"Ticketmaster",  u:74, featured:false, sold_out:false },
  { id:7,  name:"BURNING MAN",       sub:"Black Rock City",            city:"Black Rock City", country:"USA",         region:"Americas",    venue:"Black Rock Desert",             date:"Aug 30 – Sep 7, 2026",  genre:["Deep House","House","Experimental"],      artists:["Robot Heart","Disorient","Many Art Cars"],                   desc:"The playa's sound camps are a world unto themselves. Robot Heart at sunrise. Deep house at 5am across cracked desert.",                                               tag:"FESTIVAL", cap:"80,000",  ticket:"Ballot",        u:91, featured:false, sold_out:false },
  { id:8,  name:"BOILER ROOM",       sub:"World Tour 2026",            city:"Global",          country:"Worldwide",   region:"Americas",    venue:"NYC · London · Tokyo · Bogotá", date:"Year-Round 2026",       genre:["Deep House","Techno","Garage","Disco"],   artists:["Various — Invite Only"],                                     desc:"The rooms change. The principle doesn't. Access by invite, ballot, or extraordinary luck.",                                                                           tag:"PARTY",    cap:"200–800", ticket:"Invite / Ballot",u:94, featured:false, sold_out:false },
  { id:9,  name:"NUITS SONORES",     sub:"Lyon Electronic Arts",       city:"Lyon",            country:"France",      region:"Europe",      venue:"Les Grandes Locos & La Sucrière",date:"May 13–17, 2026",      genre:["Acid House","Techno","Rave","Dub"],       artists:["Leftfield","808 State","Amelie Lens","Rødhåd","Baalti"],    desc:"23rd edition. Five days across Lyon's industrial heritage sites. Acid house, rave, dub — and what the French have made of all of it.",                               tag:"FESTIVAL", cap:"20,000",  ticket:"RA",            u:91, featured:true,  sold_out:false },
  { id:10, name:"HOUGHTON",          sub:"Craig Richards' Festival",   city:"Norfolk",         country:"UK",          region:"Europe",      venue:"Houghton Hall Estate",          date:"Aug 6–9, 2026",         genre:["Deep House","Techno","Experimental"],     artists:["Craig Richards","Floating Points","Joy Orbison","Hunee"],    desc:"Sound-system obsessed, capped at 5,000. No headliner billing. The most serious festival in England.",                                                                 tag:"FESTIVAL", cap:"5,000",   ticket:"RA",            u:97, featured:false, sold_out:false },
  { id:11, name:"DEKMANTEL",         sub:"Amsterdam",                  city:"Amsterdam",       country:"Netherlands", region:"Europe",      venue:"Amsterdamse Bos",               date:"Jul 30 – Aug 3, 2026",  genre:["Deep House","Techno","Ambient"],          artists:["Omar S","Peggy Gou","Hunee","DJ Stingray","Red Axes"],       desc:"Five days in the Amsterdam forest. 2026 adds an 'At Dawn' morning series for sonic exploration.",                                                                    tag:"FESTIVAL", cap:"12,000",  ticket:"Ticketmaster",  u:88, featured:false, sold_out:false },
  { id:12, name:"FIELD DAY",         sub:"London",                     city:"London",          country:"UK",          region:"Europe",      venue:"Trent Country Park",            date:"Jun 6, 2026",           genre:["Deep House","Bass","Garage"],             artists:["Joy Orbison","Floating Points","Honey Dijon","Eliza Rose"],  desc:"Four stages. Joy Orbison, Floating Points, Honey Dijon on one bill — a masterclass in how London does bass-forward house.",                                          tag:"FESTIVAL", cap:"15,000",  ticket:"DICE",          u:82, featured:false, sold_out:false },
  { id:13, name:"JUNCTION 2",        sub:"London",                     city:"London",          country:"UK",          region:"Europe",      venue:"Under London Bridge Arches",    date:"Jun 6–7, 2026",         genre:["Techno","Electro","House"],               artists:["Blawan","Paula Temple","Phase Fatale","Shackleton"],         desc:"Raw concrete arches under London Bridge. Zero natural light. The most respected underground festival in the UK.",                                                     tag:"FESTIVAL", cap:"4,000",   ticket:"RA",            u:96, featured:false, sold_out:false },
  { id:14, name:"FABRIC",            sub:"London",                     city:"London",          country:"UK",          region:"Europe",      venue:"Fabric",                        date:"Every Weekend",          genre:["Tech House","Techno","Deep House"],       artists:["Craig Richards","Terry Francis","Blawan","Michael Bibi"],    desc:"Three rooms, a sprung concrete dancefloor you feel in your chest, 25 years of uncompromising programming.",                                                           tag:"CLUB",     cap:"1,500",   ticket:"DICE",          u:93, featured:false, sold_out:false },
  { id:15, name:"FOLD",              sub:"London",                     city:"London",          country:"UK",          region:"Europe",      venue:"FOLD, Canning Town",             date:"Every Weekend",          genre:["Techno","Tech House","Industrial"],       artists:["Objekt","Blawan","Call Super","Shackleton"],                 desc:"East London's most uncompromising room. Industrial estate, pitch-black dancefloor, no-photo policy.",                                                                tag:"CLUB",     cap:"600",     ticket:"DICE",          u:97, featured:false, sold_out:false },
  { id:16, name:"WAREHOUSE PROJECT", sub:"Manchester",                 city:"Manchester",      country:"UK",          region:"Europe",      venue:"Depot Mayfield",                date:"Sep–Dec 2026",          genre:["Tech House","Techno","House"],            artists:["Fisher","Four Tet","Peggy Gou","Solomun","Jamie Jones"],     desc:"Manchester's cathedral of electronic music. A decommissioned railway depot. Autumn's biggest UK club event.",                                                        tag:"CLUB",     cap:"10,000",  ticket:"DICE",          u:79, featured:false, sold_out:false },
  { id:17, name:"SUB CLUB",          sub:"Glasgow",                    city:"Glasgow",         country:"UK",          region:"Europe",      venue:"Sub Club",                      date:"Every Weekend",          genre:["Deep House","Techno","Garage"],           artists:["Harri & Domenic","Various Residents"],                       desc:"Open since 1987. The UK's longest-running club. Harri & Domenic still play Subculture on Saturdays.",                                                                tag:"CLUB",     cap:"410",     ticket:"Door Only",     u:98, featured:false, sold_out:false },
  { id:18, name:"TRESOR",            sub:"Berlin",                     city:"Berlin",          country:"Germany",     region:"Europe",      venue:"Tresor (Power Station)",        date:"Every Weekend",          genre:["Techno","Electro","Industrial"],          artists:["Jeff Mills","Juan Atkins","Joey Beltram"],                   desc:"Born 1991 in the vault of a demolished department store. Now in a power station. One of electronic music's sacred institutions.",                                     tag:"CLUB",     cap:"1,200",   ticket:"Door Only",     u:98, featured:false, sold_out:false },
  { id:19, name:"BERGHAIN",          sub:"Panorama Bar · Berlin",      city:"Berlin",          country:"Germany",     region:"Europe",      venue:"Berghain",                      date:"Every Weekend",          genre:["Techno","Deep House","Minimal"],          artists:["Resident Advisors","Boris","Len Faki","Tama Sumo"],          desc:"The door is the legend. Panorama Bar on Sunday — deep house, disco, daylight through frosted windows.",                                                              tag:"CLUB",     cap:"1,500",   ticket:"Door Only",     u:99, featured:false, sold_out:false },
  { id:20, name:"WATERGATE",         sub:"Berlin",                     city:"Berlin",          country:"Germany",     region:"Europe",      venue:"Watergate",                     date:"Every Weekend",          genre:["Tech House","Deep House","Minimal"],      artists:["Âme","Dixon","Recondite","Marcel Dettmann"],                 desc:"On the Spree, floor-to-ceiling windows facing the water. Âme and Dixon's home. One of Berlin's most beautiful rooms.",                                              tag:"CLUB",     cap:"700",     ticket:"Door Only",     u:95, featured:false, sold_out:false },
  { id:21, name:"CLUB DER VISION.",  sub:"Berlin",                     city:"Berlin",          country:"Germany",     region:"Europe",      venue:"Club der Visionaere",            date:"Every Weekend (Summer)", genre:["Minimal","Deep House","Techno"],          artists:["Butch","DJ Koze","Sonja Moonear","Tama Sumo"],               desc:"A floating raft on the canal in Treptow. Sunday afternoons in summer. Berlin's most intimate outdoor dancefloor.",                                                    tag:"CLUB",     cap:"400",     ticket:"Door Only",     u:97, featured:false, sold_out:false },
  { id:22, name:"ROBERT JOHNSON",    sub:"Frankfurt",                  city:"Offenbach",       country:"Germany",     region:"Europe",      venue:"Robert Johnson",                date:"Every Weekend",          genre:["Minimal","Deep House","Techno"],          artists:["Zip","Villalobos","Âme"],                                    desc:"Offenbach's legendary room. Zip is resident. It inspired a generation of minimal house producers across Europe.",                                                     tag:"CLUB",     cap:"400",     ticket:"Door Only",     u:99, featured:false, sold_out:false },
  { id:23, name:"REX CLUB",          sub:"Paris",                      city:"Paris",           country:"France",      region:"Europe",      venue:"Le Rex Club",                   date:"Every Weekend",          genre:["Techno","Deep House","Minimal"],          artists:["Laurent Garnier","DJ Stingray","Joris Voorn"],               desc:"Paris's temple of electronic music since 1990. Laurent Garnier's home. One room, one of the finest rigs in Europe.",                                                 tag:"CLUB",     cap:"800",     ticket:"Door Only",     u:96, featured:false, sold_out:false },
  { id:24, name:"CONCRETE",          sub:"Paris",                      city:"Paris",           country:"France",      region:"Europe",      venue:"Concrete",                      date:"Every Weekend",          genre:["Techno","Minimal","House"],               artists:["DJ Nobu","Call Super","Blawan","Shackleton"],                desc:"Under a Paris bridge on the Seine. Daytime raves that bleed into morning. No windows. No regrets.",                                                                  tag:"CLUB",     cap:"500",     ticket:"RA",            u:96, featured:false, sold_out:false },
  { id:25, name:"TIME WARP",         sub:"Mannheim",                   city:"Mannheim",        country:"Germany",     region:"Europe",      venue:"Maimarkthalle",                 date:"Apr 4–5, 2026",         genre:["Techno","Tech House","Minimal"],          artists:["Sven Väth","Richie Hawtin","Joris Voorn","Luciano"],         desc:"The godfather of European techno festivals since 1994. Industrial halls into sonic cathedrals.",                                                                     tag:"FESTIVAL", cap:"18,000",  ticket:"Ticketmaster",  u:80, featured:false, sold_out:false },
  { id:26, name:"HORST",             sub:"Arts & Music · Belgium",     city:"Vilvoorde",       country:"Belgium",     region:"Europe",      venue:"Asiat Park (Former Military)",  date:"May 29–31, 2026",       genre:["Techno","House","Experimental"],          artists:["Objekt","Phase Fatale","Blawan","Amnesia Scanner"],          desc:"A decommissioned military complex turned living laboratory. 2026 theme: '(R)evolution over repetition.'",                                                             tag:"FESTIVAL", cap:"8,000",   ticket:"RA",            u:95, featured:false, sold_out:false },
  { id:27, name:"LOVE INTERNATIONAL",sub:"Croatia",                    city:"Tisno",           country:"Croatia",     region:"Europe",      venue:"The Garden, Tisno",             date:"Jul 2–8, 2026",         genre:["Deep House","Disco","Balearic","Garage"], artists:["DJ Harvey","Horse Meat Disco","Hunee","Antal"],              desc:"A week on the Dalmatian coast. Boat parties, beach stages, olive groves at night. The most joyful festival in Europe.",                                               tag:"FESTIVAL", cap:"3,000",   ticket:"RA",            u:95, featured:false, sold_out:false },
  { id:28, name:"DIMENSIONS",        sub:"Croatia",                    city:"Pula",            country:"Croatia",     region:"Europe",      venue:"Fort Punta Christo",            date:"Sep 3–7, 2026",         genre:["Techno","Deep House","Experimental"],     artists:["Objekt","Four Tet","Shackleton","Ben UFO","Actress"],         desc:"A 19th-century fortress on the Adriatic. Underground tunnels, open-air stages, and the most forward-thinking bill in festival season.",                               tag:"FESTIVAL", cap:"5,000",   ticket:"RA",            u:97, featured:false, sold_out:false },
  { id:29, name:"SONAR",             sub:"Barcelona",                  city:"Barcelona",       country:"Spain",       region:"Europe",      venue:"Fira Gran Via",                 date:"Jun 18–20, 2026",       genre:["Deep House","Experimental","Electronic"], artists:["Four Tet","Floating Points","DJ Koze","Nina Kraviz"],        desc:"Where music, creativity and technology converge. SonarLab by day is as essential as the clubs at night.",                                                            tag:"FESTIVAL", cap:"120,000", ticket:"DICE",          u:70, featured:true,  sold_out:false },
  { id:30, name:"AFRONATION",        sub:"Portugal",                   city:"Portimão",        country:"Portugal",    region:"Europe",      venue:"Portimão Arena",                date:"Jul 3–5, 2026",         genre:["Afro House","Amapiano","Afrobeats"],      artists:["Black Coffee","Themba","Enoo Napa","Culoe De Song"],         desc:"The world's largest Afrobeats and Afro house gathering. The global African diaspora united on the Algarve coast.",                                                   tag:"FESTIVAL", cap:"35,000",  ticket:"Ticketmaster",  u:65, featured:false, sold_out:false },
  { id:31, name:"PARADISE",          sub:"Jamie Jones · DC-10",        city:"Ibiza",           country:"Spain",       region:"Europe",      venue:"DC-10",                         date:"Every Wed, Jun–Sep",    genre:["Tech House","Deep House"],               artists:["Jamie Jones","Lee Foss","Richy Ahmed","Heidi"],              desc:"Every Wednesday. 15th season. The world's most watched tech house residency.",                                                                                        tag:"RESIDENCY",cap:"1,500",   ticket:"RA",            u:78, featured:false, sold_out:false },
  { id:32, name:"CIRCOLOCO",         sub:"Monday · DC-10",             city:"Ibiza",           country:"Spain",       region:"Europe",      venue:"DC-10",                         date:"Every Mon, Jun–Sep",    genre:["Tech House","Minimal","Techno"],          artists:["Seth Troxler","Luciano","Ricardo Villalobos","Solomun"],     desc:"The Monday pilgrimage. An airplane hangar in the Ibiza countryside. The world's most devoted underground crowd.",                                                     tag:"RESIDENCY",cap:"1,500",   ticket:"RA",            u:85, featured:false, sold_out:false },
  { id:33, name:"RAINBOW DISCO CLUB",sub:"Japan",                      city:"Izu Peninsula",   country:"Japan",       region:"Asia-Pacific",venue:"Oku-Izu Nature Park",           date:"Apr 30 – May 3, 2026",  genre:["Deep House","Disco","Balearic"],          artists:["Hunee","DJ Harvey","Antal","Palms Trax"],                    desc:"Japan's most beloved boutique festival. A Balearic pilgrimage in the mountains. Tickets vanish in seconds.",                                                         tag:"FESTIVAL", cap:"3,000",   ticket:"RA",            u:99, featured:false, sold_out:true  },
  { id:34, name:"CONTACT",           sub:"Tokyo",                      city:"Tokyo",           country:"Japan",       region:"Asia-Pacific",venue:"Contact Shibuya",               date:"Every Weekend",          genre:["Deep House","Techno","Minimal"],          artists:["Wata Igarashi","Mars89","Romy Mats"],                        desc:"Shibuya's iconic basement bunker. Boiler Room's Tokyo home. The queue is longer than the club — worth every minute.",                                                 tag:"CLUB",     cap:"300",     ticket:"Door Only",     u:99, featured:false, sold_out:false },
  { id:35, name:"OHJO",              sub:"Tokyo",                      city:"Tokyo",           country:"Japan",       region:"Asia-Pacific",venue:"OHJO Kabukicho",                date:"Every Weekend",          genre:["Techno","House","Hard Techno"],           artists:["Layton Giordani","999999999","Paolo Ferrara"],               desc:"The red-brick castle of Kabukicho. Three floors. Asia's fastest-rising underground venue.",                                                                           tag:"CLUB",     cap:"500",     ticket:"Door Only",     u:95, featured:false, sold_out:false },
  { id:36, name:"WOMB",              sub:"Tokyo",                      city:"Tokyo",           country:"Japan",       region:"Asia-Pacific",venue:"Womb, Shibuya",                 date:"Every Weekend",          genre:["Techno","Tech House","Deep House"],       artists:["Satoshi Tomiie","Ken Ishii","Fumiya Tanaka","DJ Nobu"],      desc:"Japan's most internationally recognised club. The rig is famous, the sound is faithful, the crowd is devoted.",                                                       tag:"CLUB",     cap:"700",     ticket:"Door Only",     u:92, featured:false, sold_out:false },
  { id:37, name:"THE OBSERVATORY",   sub:"Ho Chi Minh City",           city:"Ho Chi Minh City",country:"Vietnam",     region:"Asia-Pacific",venue:"The Observatory",               date:"Every Weekend",          genre:["Deep House","Disco","Techno"],            artists:["Anh Vy","Various Residents"],                                desc:"Southeast Asia's most important underground venue. Rooftop. Intimate. Regular Boiler Room stop.",                                                                     tag:"CLUB",     cap:"400",     ticket:"Door Only",     u:97, featured:false, sold_out:false },
  { id:38, name:"MODULE",            sub:"Dubai",                      city:"Dubai",           country:"UAE",         region:"Asia-Pacific",venue:"Module, Alserkal Avenue",       date:"Every Weekend (Oct–Apr)",genre:["Tech House","Deep House","Minimal"],      artists:["Various International","Local UAE Scene"],                   desc:"Alserkal Avenue's arts hub hides the Middle East's best underground room. October to April season.",                                                                  tag:"CLUB",     cap:"600",     ticket:"DICE",          u:84, featured:false, sold_out:false },
  { id:39, name:"CAPE TOWN HOUSE",   sub:"Afro House Conference",      city:"Cape Town",       country:"South Africa",region:"Africa",      venue:"The Old Biscuit Mill",           date:"Sep 12–13, 2026",       genre:["Afro House","Deep House","Soulful House"],artists:["Black Coffee","Culoe De Song","Manoo","Themba"],             desc:"The spiritual home of Afro house. Cape Town's most important gathering for heads.",                                                                                   tag:"FESTIVAL", cap:"3,000",   ticket:"EventBrite",    u:90, featured:false, sold_out:false },
  { id:40, name:"BRIDGES FOR MUSIC", sub:"Cape Town",                  city:"Cape Town",       country:"South Africa",region:"Africa",      venue:"Langa Township",                date:"Nov 2026",              genre:["Afro House","Amapiano","Deep House"],     artists:["Bridges for Music Academy","International Guests"],          desc:"The DJ Mag-partnered music academy in Langa township. Where the next generation of Afro house producers are being shaped.",                                           tag:"FESTIVAL", cap:"2,000",   ticket:"EventBrite",    u:96, featured:false, sold_out:false },
  { id:41, name:"ADE",               sub:"Amsterdam Dance Event",      city:"Amsterdam",       country:"Netherlands", region:"Europe",      venue:"Citywide — 200+ Venues",        date:"Oct 14–18, 2026",       genre:["Tech House","Deep House","Techno"],       artists:["Full World Lineup TBA"],                                     desc:"The world's largest club music conference. 400,000 visitors, 2,500 artists, 200+ venues. Five days that define the year.",                                            tag:"FESTIVAL", cap:"400,000", ticket:"RA",            u:76, featured:false, sold_out:false },
  { id:42, name:"TOMORROWLAND",      sub:"Belgium",                    city:"Boom",            country:"Belgium",     region:"Europe",      venue:"De Schorre",                    date:"Jul 17–19 & 24–26, 2026",genre:["House","Tech House","EDM"],               artists:["David Guetta","Martin Garrix","FISHER","Keinemusik"],        desc:"400,000 from 231 countries. Two weekends. The most-recognised festival brand on earth.",                                                                              tag:"FESTIVAL", cap:"400,000", ticket:"Ticketmaster",  u:35, featured:false, sold_out:true  },
];

const TIERS = [
  { id:"free",    name:"GUEST",    price:"FREE", freq:"",       desc:"A taste. Just enough to know what you're missing.",    color:C.stone,  cta:"Enter as Guest",          features:[`${GUEST_LIMIT} events visible`,"Region & genre filters","Underground rating"],                                                                     locked:["Full 42-event database","Club & residency intel","DJ Mag Top 100"] },
  { id:"member",  name:"MEMBER",   price:"$12",  freq:"/month", desc:"Full access. Every event, every detail, every source.", color:C.gold,   cta:"Subscribe — $12 / mo",    features:["Full 42-event database","All regions, genres & types","Capacity & ticket sources","DJ Mag Top 100 panel","New drops every month"],         locked:["Private Telegram","Monthly curation email"] },
  { id:"founding",name:"FOUNDING", price:"$79",  freq:"/year",  desc:"Built for the committed. Founding status, forever.",    color:C.goldHi, cta:"Become a Founder — $79 / yr",features:["Everything in Member","Private Telegram drop channel","Monthly curation email","Early-access announcements","No price increase — ever"], locked:[], badge:"BEST VALUE" },
];

const TOP_DJS = [
  { rank:7,  name:"FISHER",       genre:"House",     award:"World's No.1 House DJ 2025" },
  { rank:17, name:"Black Coffee", genre:"Afro House", award:"Top 20 · Up 7 places" },
  { rank:20, name:"Keinemusik",   genre:"Deep House", award:"Up 15 places" },
  { rank:41, name:"Dom Dolla",    genre:"Tech House", award:"Up 25 places" },
  { rank:46, name:"John Summit",  genre:"Tech House", award:"Up 24 places" },
  { rank:48, name:"Michael Bibi", genre:"Tech House", award:"Highest New Entry" },
  { rank:55, name:"Solomun",      genre:"Deep House", award:"Highest Climber +33" },
  { rank:97, name:"Honey Dijon",  genre:"House",      award:"Re-Entry" },
];

const TRACKS = [
  { id:"kNgAMFPOKnc", title:"Home Brewed 004",     artist:"Black Coffee",   label:"HBTV"        },
  { id:"dX3VSPDNnMQ", title:"Boiler Room Berlin",   artist:"Floating Points",label:"Boiler Room" },
  { id:"6oKOVl7XaO4", title:"Boiler Room Detroit",  artist:"DJ Stingray",   label:"Boiler Room" },
  { id:"bGCSS4EQKZQ", title:"Essential Mix",        artist:"Four Tet",      label:"BBC Radio 1" },
  { id:"rX5PZp5IFvE", title:"fabric 100",           artist:"Craig Richards", label:"Fabric"      },
];

const GENRES_F  = ["All","Tech House","Deep House","Techno","Afro House","Acid House","Minimal","Disco","Balearic","Garage"];
const REGIONS_F = ["All","Americas","Europe","Asia-Pacific","Africa"];
const TAGS_F    = ["All","FESTIVAL","CLUB","RESIDENCY","PARTY"];
const TICKETS_F = ["All","DICE","RA","Ticketmaster","EventBrite","Door Only","Invite / Ballot"];

/* ─── SHARED ATOMS ───────────────────────────────────────────────────────── */
function Grain() {
  return (
    <div style={{ position:"fixed",inset:"-50%",opacity:0.055,backgroundImage:`url("data:image/svg+xml,%3Csvg viewBox='0 0 200 200' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='n'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.85' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23n)'/%3E%3C/svg%3E")`,backgroundRepeat:"repeat",animation:"grain 0.35s steps(1) infinite",pointerEvents:"none",zIndex:1 }}/>
  );
}

function VideoBg({ intensity=0.92 }) {
  return (
    <div style={{ position:"fixed",inset:0,zIndex:0,overflow:"hidden",pointerEvents:"none" }}>
      <iframe src="https://www.youtube.com/embed/zGdKzQDJGjI?autoplay=1&mute=1&loop=1&playlist=zGdKzQDJGjI&controls=0&showinfo=0&rel=0&disablekb=1&modestbranding=1&playsinline=1" title="bg" allow="autoplay;encrypted-media"
        style={{ position:"absolute",top:"50%",left:"50%",transform:"translate(-50%,-50%) scale(1.06)",width:"177.8vh",minWidth:"100%",height:"56.25vw",minHeight:"100%",border:"none",pointerEvents:"none" }}/>
      <div style={{ position:"absolute",inset:0,background:`rgba(8,7,10,${intensity})` }}/>
      <div style={{ position:"absolute",inset:0,background:"radial-gradient(ellipse at center,transparent 30%,rgba(4,3,6,0.55) 100%)" }}/>
    </div>
  );
}

function Dot({ color=C.gold, size=7 }) {
  return (
    <div style={{ position:"relative",width:size,height:size,flexShrink:0 }}>
      <div style={{ position:"absolute",inset:0,borderRadius:"50%",background:color,animation:"ringPop 2.4s infinite" }}/>
      <div style={{ position:"absolute",inset:"1px",borderRadius:"50%",background:color }}/>
    </div>
  );
}

function Ruler({ val, color=C.gold }) {
  return (
    <div style={{ display:"flex",alignItems:"center",gap:8 }}>
      <div style={{ flex:1,height:1,background:C.border }}>
        <div style={{ width:`${val}%`,height:"100%",background:`linear-gradient(90deg,${color}60,${color})` }}/>
      </div>
      <span style={{ ...mono,fontSize:8,color:C.dim,minWidth:20,textAlign:"right" }}>{val}</span>
    </div>
  );
}

function Pill({ children, active, onClick, color, sm }) {
  return (
    <button onClick={onClick} style={{ ...mono,fontSize:sm?8:9,letterSpacing:1.5,padding:sm?"3px 9px":"5px 13px",borderRadius:"100px",background:active?`${color||C.gold}18`:"transparent",color:active?(color||C.gold):C.stone,border:`1px solid ${active?(color||C.gold)+"50":C.vdim}`,transition:"all 0.2s",whiteSpace:"nowrap",flexShrink:0 }}>
      {children}
    </button>
  );
}

/* ─── PAYMENT FORM ───────────────────────────────────────────────────────── */
function PaymentForm({ tier, onSuccess, onBack }) {
  const [form, setForm] = useState({ email:"",card:"",expiry:"",cvc:"",name:"" });
  const [errs, setErrs] = useState({});
  const [foc,  setFoc]  = useState(null);
  const [busy, setBusy] = useState(false);
  const [done, setDone] = useState(false);

  const set = (k,v) => { setForm(f=>({...f,[k]:v})); setErrs(e=>({...e,[k]:null})); };
  const fmtCard = v => v.replace(/\D/g,"").slice(0,16).replace(/(.{4})/g,"$1 ").trim();
  const fmtExp  = v => { const d=v.replace(/\D/g,"").slice(0,4); return d.length>2?d.slice(0,2)+"/"+d.slice(2):d; };
  const brand = (() => { const n=form.card.replace(/\s/g,"")[0]; if(n==="4")return{l:"VISA",c:"#C8A84B"}; if(n==="5")return{l:"MC",c:"#C46444"}; if(n==="3")return{l:"AMEX",c:"#5A82B0"}; return null; })();
  const cardDisplay = (form.card.replace(/\s/g,"").padEnd(16,"·").match(/.{1,4}/g)||[]).join(" ");
  const ac = tier.id==="founding"?C.goldHi:C.gold;

  const validate = () => {
    const e={};
    if(!form.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) e.email="Valid email required";
    if(form.card.replace(/\s/g,"").length<16)            e.card ="Complete card number";
    if(!form.expiry.match(/^\d{2}\/\d{2}$/))             e.expiry="MM/YY";
    if(form.cvc.length<3)                                e.cvc  ="3+ digits";
    if(!form.name.trim())                                e.name ="Required";
    setErrs(e); return Object.keys(e).length===0;
  };

  const pay = () => { if(!validate()) return; setBusy(true); setTimeout(()=>{ setDone(true); setTimeout(onSuccess,1300); },2200); };

  const inp = k => ({
    width:"100%", background:"rgba(12,10,8,0.75)", border:`1px solid ${errs[k]?C.err+"80":foc===k?ac+"55":C.border}`,
    borderRadius:"4px", padding:"12px 14px", ...mono, fontSize:13, color:C.ivory, transition:"border-color 0.2s",
  });

  if(done) return (
    <div style={{ textAlign:"center",padding:"32px 0" }}>
      <div style={{ width:48,height:48,borderRadius:"50%",border:`1px solid ${C.ok}60`,background:`${C.ok}12`,display:"flex",alignItems:"center",justifyContent:"center",margin:"0 auto 16px",fontSize:20 }}>✓</div>
      <div style={{ ...display,fontSize:28,color:C.ok,fontWeight:600 }}>Access Granted</div>
      <div style={{ ...mono,fontSize:10,color:C.stone,marginTop:8 }}>Welcome to Bassline {tier.name}.</div>
    </div>
  );

  const LR = ({label,err}) => (
    <div style={{ display:"flex",justifyContent:"space-between",marginBottom:6 }}>
      <span style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:2.5 }}>{label}</span>
      {err&&<span style={{ ...mono,fontSize:8,color:C.err }}>{err}</span>}
    </div>
  );

  return (
    <div>
      {/* Card preview */}
      <div style={{ background:"linear-gradient(135deg,rgba(20,16,10,0.9),rgba(28,22,12,0.9))",border:`1px solid ${ac}25`,borderRadius:"8px",padding:"16px 18px",marginBottom:20 }}>
        <div style={{ display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:16 }}>
          <div style={{ ...display,fontSize:12,color:ac,letterSpacing:4,fontStyle:"italic" }}>Bassline</div>
          {brand?<span style={{ ...mono,fontSize:9,color:brand.c }}>{brand.l}</span>:<span style={{ ...mono,fontSize:8,color:C.vdim }}>CARD</span>}
        </div>
        <div style={{ ...mono,fontSize:13,color:form.card?C.ivory:C.dim,letterSpacing:3,marginBottom:12,fontWeight:300 }}>{cardDisplay}</div>
        <div style={{ display:"flex",justifyContent:"space-between" }}>
          <div><div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2,marginBottom:2 }}>CARDHOLDER</div><div style={{ ...mono,fontSize:10,color:form.name?C.stone:C.dim }}>{form.name||"YOUR NAME"}</div></div>
          <div style={{ textAlign:"right" }}><div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2,marginBottom:2 }}>EXPIRES</div><div style={{ ...mono,fontSize:10,color:form.expiry?C.stone:C.dim }}>{form.expiry||"MM/YY"}</div></div>
        </div>
      </div>

      <div style={{ marginBottom:12 }}><LR label="EMAIL" err={errs.email}/><input value={form.email} onChange={e=>set("email",e.target.value)} onFocus={()=>setFoc("email")} onBlur={()=>setFoc(null)} placeholder="you@email.com" type="email" style={inp("email")}/></div>
      <div style={{ marginBottom:12 }}>
        <LR label="CARD NUMBER" err={errs.card}/>
        <div style={{ position:"relative" }}>
          <input value={form.card} onChange={e=>set("card",fmtCard(e.target.value))} onFocus={()=>setFoc("card")} onBlur={()=>setFoc(null)} placeholder="0000  0000  0000  0000" maxLength={19} style={{ ...inp("card"),paddingRight:brand?50:14 }}/>
          {brand&&<span style={{ position:"absolute",right:12,top:"50%",transform:"translateY(-50%)",...mono,fontSize:9,color:brand.c }}>{brand.l}</span>}
        </div>
      </div>
      <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:12 }}>
        <div><LR label="EXPIRY" err={errs.expiry}/><input value={form.expiry} onChange={e=>set("expiry",fmtExp(e.target.value))} onFocus={()=>setFoc("expiry")} onBlur={()=>setFoc(null)} placeholder="MM / YY" maxLength={5} style={inp("expiry")}/></div>
        <div><LR label="CVC" err={errs.cvc}/><input value={form.cvc} onChange={e=>set("cvc",e.target.value.replace(/\D/g,"").slice(0,4))} onFocus={()=>setFoc("cvc")} onBlur={()=>setFoc(null)} placeholder="···" maxLength={4} type="password" style={inp("cvc")}/></div>
      </div>
      <div style={{ marginBottom:18 }}><LR label="NAME ON CARD" err={errs.name}/><input value={form.name} onChange={e=>set("name",e.target.value.toUpperCase())} onFocus={()=>setFoc("name")} onBlur={()=>setFoc(null)} placeholder="FULL NAME" style={inp("name")}/></div>

      <button onClick={pay} disabled={busy} style={{ width:"100%",padding:"15px",background:busy?"transparent":`linear-gradient(135deg,${ac},${ac}CC)`,color:busy?C.dim:"#0A0806",border:`1px solid ${busy?C.border:ac}`,borderRadius:"4px",...display,fontSize:18,fontWeight:600,letterSpacing:3,transition:"all 0.22s",display:"flex",alignItems:"center",justifyContent:"center",gap:10 }}>
        {busy?<><span style={{ width:12,height:12,border:`1.5px solid ${C.dim}`,borderTop:`1.5px solid ${C.stone}`,borderRadius:"50%",display:"inline-block",animation:"spin 0.9s linear infinite" }}/> Processing…</>:`Pay ${tier.price}${tier.freq} →`}
      </button>

      <div style={{ display:"flex",gap:16,justifyContent:"center",marginTop:14,flexWrap:"wrap" }}>
        {[["🔒","SSL Secure"],["↩","Cancel anytime"],["⚡","Instant access"]].map(([ic,tx])=>(
          <div key={tx} style={{ display:"flex",gap:5,alignItems:"center" }}>
            <span style={{ fontSize:10 }}>{ic}</span>
            <span style={{ ...mono,fontSize:8,color:C.dim }}>{tx}</span>
          </div>
        ))}
      </div>
      {onBack&&<button onClick={onBack} style={{ display:"block",margin:"14px auto 0",background:"none",border:"none",...mono,fontSize:8,color:C.dim,letterSpacing:2,textDecoration:"underline" }}>← back to plans</button>}
    </div>
  );
}

/* ─── SCREEN 1: PASSWORD GATE ────────────────────────────────────────────── */
function PasswordGate({ onSuccess }) {
  const [val, setVal]   = useState("");
  const [err, setErr]   = useState(false);
  const [tick, setTick] = useState(0);
  useEffect(()=>{ const i=setInterval(()=>setTick(t=>t+1),80); return()=>clearInterval(i); },[]);

  const attempt = () => {
    if(val.trim().toUpperCase()===PASSWORD) onSuccess();
    else { setErr(true); setTimeout(()=>setErr(false),2000); }
  };

  return (
    <div style={{ minHeight:"100vh",display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",padding:"24px 20px",position:"relative" }}>
      <VideoBg intensity={0.84}/>
      <Grain/>
      <div style={{ position:"relative",zIndex:2,width:"100%",maxWidth:360,textAlign:"center",animation:"fadeIn 0.8s ease" }}>
        {/* Waveform */}
        <div style={{ display:"flex",justifyContent:"center",alignItems:"flex-end",gap:3,height:28,marginBottom:36 }}>
          {[7,13,5,18,4,14,9,16,6,12,7].map((h,i)=>(
            <div key={i} style={{ width:2,borderRadius:1,background:C.gold,height:`${Math.max(3,h*0.45+Math.abs(Math.sin((tick+i*5)*0.25))*h*0.55)}px`,transition:"height 0.08s ease",opacity:0.55+Math.abs(Math.sin((tick+i)*0.18))*0.45 }}/>
          ))}
        </div>
        {/* Wordmark */}
        <div style={{ ...display,fontSize:"clamp(46px,12vw,100px)",color:C.ivory,lineHeight:1,fontWeight:300,letterSpacing:"clamp(6px,2vw,12px)",marginBottom:6 }}>BASSLINE</div>
        <div style={{ ...mono,fontSize:"clamp(7px,2vw,8px)",color:C.goldDim,letterSpacing:"clamp(3px,1.5vw,5px)",marginBottom:40 }}>GLOBAL HOUSE INDEX · MEMBERS ONLY</div>
        <div style={{ width:1,height:36,background:`linear-gradient(${C.gold}60,transparent)`,margin:"0 auto 36px" }}/>
        <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:3,marginBottom:10 }}>ENTER ACCESS CODE</div>
        <input value={val} onChange={e=>{ setVal(e.target.value.toUpperCase()); setErr(false); }} onKeyDown={e=>e.key==="Enter"&&attempt()} maxLength={20} placeholder="· · · · · · · ·" type="password"
          style={{ width:"100%",background:"rgba(10,8,6,0.65)",border:`1px solid ${err?C.err+"70":C.border}`,borderRadius:"4px",padding:"15px",...mono,fontSize:20,color:C.ivory,letterSpacing:8,textAlign:"center",backdropFilter:"blur(20px)",transition:"border-color 0.2s",marginBottom:10 }}/>
        {err&&<div style={{ ...mono,fontSize:8,color:C.err,letterSpacing:2,marginBottom:8 }}>INCORRECT CODE · TRY AGAIN</div>}
        <button onClick={attempt} style={{ width:"100%",background:`linear-gradient(135deg,${C.gold}E0,${C.gold}A0)`,border:"none",borderRadius:"4px",padding:"15px",...display,fontSize:20,fontWeight:600,color:"#0A0806",letterSpacing:5 }}
          onMouseEnter={e=>e.currentTarget.style.opacity="0.82"} onMouseLeave={e=>e.currentTarget.style.opacity="1"}>
          Request Access
        </button>
        <div style={{ ...mono,fontSize:8,color:C.vdim,marginTop:18,letterSpacing:2,lineHeight:2 }}>
          BY ENTERING YOU AGREE TO OUR TERMS<br/>
          <span style={{ color:C.goldDim }}>HINT: THE ANSWER IS IN THE NAME</span>
        </div>
      </div>
      <div style={{ position:"fixed",bottom:76,...mono,fontSize:7,color:C.vdim,letterSpacing:3,zIndex:2 }}>© 2026 BASSLINE · ALL RIGHTS RESERVED</div>
    </div>
  );
}

/* ─── SCREEN 2: PRICING ──────────────────────────────────────────────────── */
function PricingScreen({ onSelect }) {
  const [checkout, setCheckout] = useState(null);
  const w = useW();
  const mob = w < 640;

  if(checkout) {
    const ac = checkout.id==="founding"?C.goldHi:C.gold;
    return (
      <div style={{ minHeight:"100vh",background:C.bg,position:"relative" }}>
        <VideoBg intensity={0.93}/>
        <Grain/>
        <div style={{ position:"relative",zIndex:2,borderBottom:`1px solid ${C.border}`,padding:"14px 20px",display:"flex",justifyContent:"space-between",alignItems:"center",backdropFilter:"blur(20px)",background:C.glass }}>
          <button onClick={()=>setCheckout(null)} style={{ background:"none",border:"none",...mono,fontSize:8,color:C.stone,letterSpacing:2 }}>← BACK</button>
          <div style={{ ...display,fontSize:13,color:C.dim,letterSpacing:5,fontStyle:"italic" }}>Bassline · Secure Checkout</div>
          <div style={{ ...mono,fontSize:8,color:C.dim }}>🔒 SSL</div>
        </div>
        <div style={{ maxWidth:880,margin:"0 auto",padding:"28px 20px",position:"relative",zIndex:2 }}>
          {mob ? (
            /* MOBILE: stacked */
            <div>
              <div style={{ background:"rgba(14,12,10,0.75)",border:`1px solid ${ac}22`,borderRadius:"8px",padding:"20px",marginBottom:20,backdropFilter:"blur(16px)" }}>
                {checkout.badge&&<div style={{ ...mono,fontSize:7,color:C.goldHi,letterSpacing:2,marginBottom:8 }}>{checkout.badge}</div>}
                <div style={{ ...display,fontSize:46,color:ac,lineHeight:1,fontWeight:300 }}>{checkout.price}<span style={{ ...mono,fontSize:13,color:C.stone }}>{checkout.freq}</span></div>
                <div style={{ ...display,fontSize:22,color:C.ivory,letterSpacing:4,margin:"4px 0 6px",fontWeight:600 }}>{checkout.name}</div>
                {checkout.id==="founding"&&<div style={{ ...mono,fontSize:8,color:C.ok }}>SAVE $65 vs monthly</div>}
                <div style={{ display:"flex",justifyContent:"space-between",alignItems:"center",borderTop:`1px solid ${C.border}`,marginTop:12,paddingTop:12 }}>
                  <span style={{ ...mono,fontSize:9,color:C.ivory }}>TOTAL TODAY</span>
                  <span style={{ ...display,fontSize:24,color:ac,fontWeight:300 }}>{checkout.price}</span>
                </div>
              </div>
              <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:3,marginBottom:14 }}>PAYMENT DETAILS</div>
              <PaymentForm tier={checkout} onSuccess={()=>onSelect(checkout.id)} onBack={()=>setCheckout(null)}/>
            </div>
          ) : (
            /* DESKTOP: side by side */
            <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:40 }}>
              <div>
                <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:3,marginBottom:18 }}>ORDER SUMMARY</div>
                <div style={{ background:"rgba(14,12,10,0.75)",border:`1px solid ${ac}22`,borderRadius:"8px",padding:"22px",marginBottom:22,backdropFilter:"blur(16px)" }}>
                  {checkout.badge&&<div style={{ ...mono,fontSize:7,color:C.goldHi,letterSpacing:2,marginBottom:8 }}>{checkout.badge}</div>}
                  <div style={{ ...display,fontSize:48,color:ac,lineHeight:1,fontWeight:300 }}>{checkout.price}<span style={{ ...mono,fontSize:13,color:C.stone }}>{checkout.freq}</span></div>
                  <div style={{ ...display,fontSize:24,color:C.ivory,letterSpacing:4,margin:"4px 0 6px",fontWeight:600 }}>{checkout.name}</div>
                  <div style={{ ...mono,fontSize:9,color:C.stone,lineHeight:1.8 }}>{checkout.desc}</div>
                </div>
                {checkout.features.map(f=>(
                  <div key={f} style={{ display:"flex",gap:10,marginBottom:8 }}>
                    <span style={{ color:ac,fontSize:10,marginTop:1 }}>✓</span>
                    <span style={{ ...mono,fontSize:9,color:C.stone }}>{f}</span>
                  </div>
                ))}
                <div style={{ borderTop:`1px solid ${C.border}`,paddingTop:14,marginTop:14 }}>
                  {checkout.id==="founding"&&<div style={{ display:"flex",justifyContent:"space-between",marginBottom:8 }}><span style={{ ...mono,fontSize:8,color:C.dim }}>vs Monthly ($144/yr)</span><span style={{ ...mono,fontSize:8,color:C.ok }}>SAVE $65</span></div>}
                  <div style={{ display:"flex",justifyContent:"space-between",alignItems:"center" }}>
                    <span style={{ ...mono,fontSize:9,color:C.ivory }}>TOTAL TODAY</span>
                    <span style={{ ...display,fontSize:26,color:ac,fontWeight:300 }}>{checkout.price}</span>
                  </div>
                </div>
              </div>
              <div>
                <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:3,marginBottom:18 }}>PAYMENT DETAILS</div>
                <PaymentForm tier={checkout} onSuccess={()=>onSelect(checkout.id)} onBack={()=>setCheckout(null)}/>
              </div>
            </div>
          )}
        </div>
      </div>
    );
  }

  return (
    <div style={{ minHeight:"100vh",position:"relative" }}>
      <VideoBg intensity={0.88}/>
      <Grain/>
      <div style={{ position:"relative",zIndex:2,padding:"56px 20px 80px" }}>
        <div style={{ maxWidth:1020,margin:"0 auto" }}>
          <div style={{ textAlign:"center",marginBottom:48 }}>
            <div style={{ ...mono,fontSize:8,color:C.goldDim,letterSpacing:4,marginBottom:14 }}>ACCESS GRANTED · CHOOSE YOUR MEMBERSHIP</div>
            <div style={{ ...display,fontSize:"clamp(36px,7vw,76px)",color:C.ivory,lineHeight:1,fontWeight:300,letterSpacing:"clamp(2px,1vw,6px)",marginBottom:16 }}>
              Choose Your<br/><span style={{ color:C.gold,fontStyle:"italic" }}>Access Level</span>
            </div>
            <p style={{ ...mono,fontSize:11,color:C.stone,maxWidth:420,margin:"0 auto",lineHeight:1.9 }}>
              Bassline indexes 42+ house music events, clubs and residencies worldwide.
            </p>
          </div>

          {/* Tier cards */}
          <div style={{ display:"grid",gridTemplateColumns:mob?"1fr":w<900?"1fr 1fr":"repeat(3,1fr)",gap:10,marginBottom:36 }}>
            {TIERS.map(tier=>(
              <div key={tier.id} style={{ background:"rgba(12,10,8,0.75)",backdropFilter:"blur(24px)",border:`1px solid ${tier.id==="founding"?C.gold+"30":C.border}`,borderRadius:"10px",padding:"26px 22px",position:"relative",overflow:"hidden" }}>
                {tier.badge&&<div style={{ position:"absolute",top:0,right:0,background:C.gold,...mono,fontSize:7,color:"#080806",padding:"5px 12px",letterSpacing:2 }}>{tier.badge}</div>}
                <div style={{ ...display,fontSize:48,color:tier.color,lineHeight:1,fontWeight:300 }}>{tier.price}<span style={{ ...mono,fontSize:13,color:C.stone }}>{tier.freq}</span></div>
                <div style={{ ...display,fontSize:22,color:C.ivory,letterSpacing:4,marginTop:4,fontWeight:600 }}>{tier.name}</div>
                <div style={{ ...mono,fontSize:10,color:C.stone,marginTop:6,marginBottom:18,lineHeight:1.8 }}>{tier.desc}</div>
                <button onClick={()=>tier.id==="free"?onSelect("free"):setCheckout(tier)}
                  style={{ width:"100%",padding:"13px",background:tier.id==="free"?"transparent":tier.id==="founding"?`linear-gradient(135deg,${C.goldHi},${C.gold})`:`linear-gradient(135deg,${C.gold},${C.goldDim}CC)`,color:tier.id==="free"?C.stone:"#0A0806",border:`1px solid ${tier.id==="free"?C.border:C.gold}`,borderRadius:"6px",...display,fontSize:17,fontWeight:600,letterSpacing:3,transition:"opacity 0.2s",marginBottom:20 }}
                  onMouseEnter={e=>e.currentTarget.style.opacity="0.8"} onMouseLeave={e=>e.currentTarget.style.opacity="1"}>
                  {tier.cta}
                </button>
                <div style={{ display:"flex",flexDirection:"column",gap:8 }}>
                  {tier.features.map(f=>(<div key={f} style={{ display:"flex",gap:8 }}><span style={{ color:tier.color,fontSize:9,marginTop:2 }}>✓</span><span style={{ ...mono,fontSize:9,color:C.stone,lineHeight:1.5 }}>{f}</span></div>))}
                  {tier.locked.map(f=>(<div key={f} style={{ display:"flex",gap:8,opacity:0.22 }}><span style={{ fontSize:9,marginTop:2 }}>—</span><span style={{ ...mono,fontSize:9,color:C.dim,lineHeight:1.5 }}>{f}</span></div>))}
                </div>
              </div>
            ))}
          </div>
          <div style={{ display:"flex",justifyContent:"center",gap:mob?16:32,flexWrap:"wrap" }}>
            {["🔒 Secure checkout","↩ Cancel anytime","⚡ Instant access"].map(t=>(
              <span key={t} style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:1.5 }}>{t}</span>
            ))}
          </div>
        </div>
      </div>
    </div>
  );
}

/* ─── SCREEN 3: MAIN APP ─────────────────────────────────────────────────── */
function MainApp({ tier, onLogout }) {
  const isPaid = tier==="member"||tier==="founding";
  const w = useW();
  const mob = w < 640;
  const tablet = w < 1024;

  const [genre,  setGenre]  = useState("All");
  const [region, setRegion] = useState("All");
  const [tag,    setTag]    = useState("All");
  const [ticket, setTicket] = useState("All");
  const [search, setSearch] = useState("");
  const [sel,    setSel]    = useState(null);
  const [djOpen, setDjOpen] = useState(false);
  const [view,   setView]   = useState("grid");
  const [tick,   setTick]   = useState(0);
  const [upsell, setUpsell] = useState(false);
  const [menuOpen, setMenuOpen] = useState(false);
  useEffect(()=>{ const i=setInterval(()=>setTick(t=>t+1),85); return()=>clearInterval(i); },[]);

  const visible = isPaid ? ALL_EVENTS : ALL_EVENTS.slice(0,GUEST_LIMIT);
  const locked  = isPaid ? [] : ALL_EVENTS.slice(GUEST_LIMIT);

  const filtered = ALL_EVENTS.filter(e=>{
    if(genre!=="All"&&!e.genre.includes(genre)) return false;
    if(region!=="All"&&e.region!==region) return false;
    if(tag!=="All"&&e.tag!==tag) return false;
    if(ticket!=="All"&&e.ticket!==ticket) return false;
    if(search){ const q=search.toLowerCase(); return e.name.toLowerCase().includes(q)||e.city.toLowerCase().includes(q)||e.artists.some(a=>a.toLowerCase().includes(q))||e.genre.some(g=>g.toLowerCase().includes(q)); }
    return true;
  });

  const visF  = filtered.filter(e=>visible.map(x=>x.id).includes(e.id));
  const lockF = filtered.filter(e=>locked.map(x=>x.id).includes(e.id));
  const featured = visF.filter(e=>e.featured);
  const rest     = visF.filter(e=>!e.featured);

  const tierColor = tier==="founding"?C.goldHi:tier==="member"?C.gold:C.stone;

  const gridCols = mob ? "1fr" : tablet ? "1fr 1fr" : "repeat(3,1fr)";
  const featCols = mob ? "1fr" : "1fr 1fr";

  return (
    <>
      <VideoBg intensity={0.95}/>
      <Grain/>

      {/* NAV */}
      <nav style={{ position:"sticky",top:0,zIndex:100,background:C.glass,backdropFilter:"blur(20px)",borderBottom:`1px solid ${C.border}` }}>
        <div style={{ maxWidth:1340,margin:"0 auto",padding:"0 16px",height:52,display:"flex",alignItems:"center",justifyContent:"space-between",gap:12 }}>
          {/* Logo */}
          <div style={{ display:"flex",alignItems:"center",gap:10 }}>
            <div style={{ display:"flex",alignItems:"flex-end",gap:2,height:16 }}>
              {[6,12,4,16,4,10,6].map((h,i)=>(
                <div key={i} style={{ width:2,borderRadius:1,background:C.gold,height:`${Math.max(3,h*0.45+Math.abs(Math.sin((tick+i*5)*0.22))*h*0.55)}px`,transition:"height 0.085s ease" }}/>
              ))}
            </div>
            <div>
              <span style={{ ...display,fontSize:16,color:C.ivory,letterSpacing:4,fontWeight:600 }}>BASSLINE</span>
              {!mob&&<span style={{ ...mono,fontSize:7,color:C.vdim,display:"block",letterSpacing:2.5 }}>GLOBAL HOUSE INDEX</span>}
            </div>
          </div>

          {/* Desktop nav items */}
          {!mob ? (
            <div style={{ display:"flex",gap:18,alignItems:"center" }}>
              {!isPaid&&<button onClick={()=>setUpsell(true)} style={{ background:`${C.gold}18`,border:`1px solid ${C.gold}40`,borderRadius:"100px",padding:"4px 12px",...mono,fontSize:8,letterSpacing:2,color:C.gold }}>UPGRADE →</button>}
              <button onClick={()=>setDjOpen(p=>!p)} style={{ background:"none",border:"none",...mono,fontSize:8,letterSpacing:2,color:djOpen?C.gold:C.stone,borderBottom:djOpen?`1px solid ${C.gold}`:"1px solid transparent",padding:"2px 0",transition:"all 0.2s" }}>DJ MAG TOP 100</button>
              <div style={{ display:"flex",gap:3 }}>
                {["grid","list"].map(v=>(
                  <button key={v} onClick={()=>setView(v)} style={{ background:view===v?`${C.gold}12`:"transparent",border:`1px solid ${view===v?C.gold+"30":C.border}`,color:view===v?C.gold:C.dim,width:26,height:24,borderRadius:"4px",display:"flex",alignItems:"center",justifyContent:"center" }}>
                    {v==="grid"?<svg width="10" height="10" viewBox="0 0 10 10"><rect x="0" y="0" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="6" y="0" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="0" y="6" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="6" y="6" width="4" height="4" fill="currentColor" rx="0.5"/></svg>:<svg width="10" height="10" viewBox="0 0 10 10"><rect x="0" y="0.5" width="10" height="1.5" fill="currentColor" rx="0.3"/><rect x="0" y="4" width="10" height="1.5" fill="currentColor" rx="0.3"/><rect x="0" y="7.5" width="10" height="1.5" fill="currentColor" rx="0.3"/></svg>}
                  </button>
                ))}
              </div>
              <div style={{ background:`${tierColor}15`,border:`1px solid ${tierColor}40`,borderRadius:"100px",padding:"3px 10px",...mono,fontSize:7,color:tierColor,letterSpacing:2 }}>{tier.toUpperCase()}</div>
              <button onClick={onLogout} style={{ background:"none",border:"none",...mono,fontSize:7,color:C.vdim,letterSpacing:2 }}>LOGOUT</button>
            </div>
          ) : (
            /* Mobile: hamburger */
            <div style={{ display:"flex",gap:10,alignItems:"center" }}>
              {!isPaid&&<button onClick={()=>setUpsell(true)} style={{ background:`${C.gold}18`,border:`1px solid ${C.gold}35`,borderRadius:"100px",padding:"4px 10px",...mono,fontSize:7,letterSpacing:2,color:C.gold }}>UPGRADE</button>}
              <button onClick={()=>setMenuOpen(p=>!p)} style={{ background:"none",border:`1px solid ${C.border}`,borderRadius:"4px",padding:"6px 10px",color:menuOpen?C.gold:C.stone,display:"flex",alignItems:"center",justifyContent:"center" }}>
                {menuOpen
                  ? <svg width="14" height="14" viewBox="0 0 14 14" fill="none"><path d="M2 2l10 10M12 2l-10 10" stroke="currentColor" strokeWidth="1.5" strokeLinecap="round"/></svg>
                  : <svg width="16" height="12" viewBox="0 0 16 12" fill="none"><rect y="0" width="16" height="1.5" rx="0.75" fill="currentColor"/><rect y="5" width="16" height="1.5" rx="0.75" fill="currentColor"/><rect y="10" width="16" height="1.5" rx="0.75" fill="currentColor"/></svg>
                }
              </button>
            </div>
          )}
        </div>

        {/* Mobile dropdown menu */}
        {mob && menuOpen && (
          <div style={{ background:"rgba(10,8,6,0.98)",backdropFilter:"blur(20px)",borderTop:`1px solid ${C.border}`,padding:"16px 20px",display:"flex",flexDirection:"column",gap:14,position:"relative",zIndex:10 }}>
            <button onClick={()=>{ setDjOpen(p=>!p); setMenuOpen(false); }} style={{ background:"none",border:"none",...mono,fontSize:10,letterSpacing:2,color:djOpen?C.gold:C.stone,textAlign:"left",padding:0 }}>DJ MAG TOP 100</button>
            <div style={{ display:"flex",gap:3 }}>
              {["grid","list"].map(v=>(
                <button key={v} onClick={()=>{ setView(v); setMenuOpen(false); }} style={{ background:view===v?`${C.gold}12`:"transparent",border:`1px solid ${view===v?C.gold+"30":C.border}`,color:view===v?C.gold:C.dim,width:34,height:30,borderRadius:"4px",display:"flex",alignItems:"center",justifyContent:"center" }}>
                  {v==="grid"?<svg width="11" height="11" viewBox="0 0 10 10"><rect x="0" y="0" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="6" y="0" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="0" y="6" width="4" height="4" fill="currentColor" rx="0.5"/><rect x="6" y="6" width="4" height="4" fill="currentColor" rx="0.5"/></svg>:<svg width="11" height="11" viewBox="0 0 10 10"><rect x="0" y="0.5" width="10" height="1.5" fill="currentColor" rx="0.3"/><rect x="0" y="4" width="10" height="1.5" fill="currentColor" rx="0.3"/><rect x="0" y="7.5" width="10" height="1.5" fill="currentColor" rx="0.3"/></svg>}
                </button>
              ))}
            </div>
            <div style={{ display:"flex",justifyContent:"space-between",alignItems:"center" }}>
              <div style={{ background:`${tierColor}15`,border:`1px solid ${tierColor}40`,borderRadius:"100px",padding:"3px 10px",...mono,fontSize:7,color:tierColor,letterSpacing:2 }}>{tier.toUpperCase()}</div>
              <button onClick={onLogout} style={{ background:"none",border:"none",...mono,fontSize:8,color:C.dim,letterSpacing:2 }}>LOGOUT</button>
            </div>
          </div>
        )}
      </nav>

      {/* TICKER */}
      <div style={{ background:`${C.gold}16`,borderBottom:`1px solid ${C.gold}18`,height:22,display:"flex",alignItems:"center",overflow:"hidden",position:"relative",zIndex:99 }}>
        <div style={{ display:"flex",whiteSpace:"nowrap",animation:"ticker 55s linear infinite" }}>
          {[...ALL_EVENTS,...ALL_EVENTS].map((e,i)=>(
            <span key={i} style={{ ...mono,fontSize:8,color:C.goldDim,padding:"0 32px",letterSpacing:1.5 }}>◆  {e.name}  ·  {e.city}  ·  {e.date}</span>
          ))}
        </div>
      </div>

      {/* DJ MAG PANEL */}
      {djOpen&&(
        <div style={{ background:C.glass,backdropFilter:"blur(20px)",borderBottom:`1px solid ${C.border}`,padding:"18px 16px",position:"relative",zIndex:98 }}>
          <div style={{ maxWidth:1340,margin:"0 auto" }}>
            <div style={{ display:"flex",justifyContent:"space-between",marginBottom:14 }}>
              <div>
                <div style={{ ...display,fontSize:17,color:C.ivory,letterSpacing:3,fontWeight:600 }}>DJ Mag Top 100 — House</div>
                <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:2,marginTop:2 }}>2025 OFFICIAL RESULTS · 231 COUNTRIES</div>
              </div>
            </div>
            <div style={{ display:"grid",gridTemplateColumns:mob?"1fr 1fr":w<900?"repeat(3,1fr)":"repeat(4,1fr)",gap:6 }}>
              {TOP_DJS.map(dj=>(
                <div key={dj.name} style={{ background:"rgba(14,12,9,0.75)",border:`1px solid ${C.border}`,borderRadius:"4px",padding:"10px 12px",display:"flex",gap:10,alignItems:"center" }}>
                  <div style={{ ...display,fontSize:22,color:C.vdim,lineHeight:1,minWidth:28,fontWeight:300 }}>#{dj.rank}</div>
                  <div>
                    <div style={{ ...display,fontSize:13,color:C.ivory,fontWeight:600 }}>{dj.name}</div>
                    <div style={{ ...mono,fontSize:7,color:C.dim }}>{dj.genre}</div>
                    <div style={{ ...mono,fontSize:7,color:C.gold }}>{dj.award}</div>
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      )}

      <div style={{ maxWidth:1340,margin:"0 auto",padding:`24px 16px 80px`,position:"relative",zIndex:2 }}>

        {/* GUEST BANNER */}
        {!isPaid&&(
          <div onClick={()=>setUpsell(true)} style={{ cursor:"pointer",background:`${C.gold}08`,border:`1px solid ${C.gold}20`,borderRadius:"8px",padding:"14px 16px",marginBottom:22,display:"flex",justifyContent:"space-between",alignItems:"center",gap:12,backdropFilter:"blur(12px)" }}>
            <div style={{ minWidth:0 }}>
              <div style={{ ...mono,fontSize:8,color:C.gold,letterSpacing:2,marginBottom:3 }}>GUEST — {GUEST_LIMIT} of {ALL_EVENTS.length} EVENTS</div>
              <div style={{ ...mono,fontSize:mob?9:10,color:C.stone }}>Upgrade for full database access.</div>
            </div>
            <div style={{ background:`${C.gold}18`,border:`1px solid ${C.gold}40`,borderRadius:"4px",padding:"7px 12px",...mono,fontSize:8,color:C.gold,letterSpacing:2,flexShrink:0 }}>UNLOCK →</div>
          </div>
        )}

        {/* STATS */}
        <div style={{ display:"grid",gridTemplateColumns:"repeat(2,1fr)",gap:1,marginBottom:36,border:`1px solid ${C.border}`,borderRadius:"8px",overflow:"hidden" }}>
          {[{l:"Events",v:ALL_EVENTS.length},{l:"Countries",v:16},{l:"Visible",v:isPaid?ALL_EVENTS.length:GUEST_LIMIT},{l:"Genres",v:14}].map(s=>(
            <div key={s.l} style={{ background:"rgba(14,12,9,0.75)",padding:"16px 18px",backdropFilter:"blur(12px)" }}>
              <div style={{ ...display,fontSize:mob?34:42,color:C.gold,lineHeight:1,fontWeight:300 }}>{s.v}</div>
              <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2,marginTop:3 }}>{s.l.toUpperCase()}</div>
            </div>
          ))}
        </div>

        {/* HERO */}
        <div style={{ marginBottom:36 }}>
          <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:4,marginBottom:10 }}>WHERE TO BE · WHEN TO BE THERE</div>
          <div style={{ ...display,fontSize:"clamp(42px,9vw,90px)",color:C.ivory,lineHeight:0.92,fontWeight:300 }}>
            The World<br/><span style={{ color:C.gold,fontStyle:"italic" }}>Is Dancing</span>
          </div>
        </div>

        {/* FILTERS — horizontal scroll on mobile */}
        <div style={{ display:"flex",flexDirection:"column",gap:10,marginBottom:28 }}>
          <div style={{ position:"relative" }}>
            <svg style={{ position:"absolute",left:12,top:"50%",transform:"translateY(-50%)",pointerEvents:"none" }} width="13" height="13" viewBox="0 0 13 13" fill="none"><circle cx="5.5" cy="5.5" r="4" stroke={C.dim} strokeWidth="1.2"/><path d="M9 9L11.5 11.5" stroke={C.dim} strokeWidth="1.2" strokeLinecap="round"/></svg>
            <input value={search} onChange={e=>setSearch(e.target.value)} placeholder="Search events, cities, artists…"
              style={{ width:"100%",background:"rgba(12,10,8,0.7)",border:`1px solid ${C.border}`,borderRadius:"4px",padding:"11px 14px 11px 34px",...mono,fontSize:12,color:C.ivory,backdropFilter:"blur(12px)" }}/>
          </div>
          {[{l:"REGION",items:REGIONS_F,s:region,fn:setRegion},{l:"TYPE",items:TAGS_F,s:tag,fn:setTag},{l:"GENRE",items:GENRES_F,s:genre,fn:setGenre},{l:"TICKETS",items:TICKETS_F,s:ticket,fn:setTicket}].map(({l,items,s,fn})=>(
            <div key={l} style={{ display:"flex",gap:6,alignItems:"center" }}>
              <span style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2,minWidth:44,flexShrink:0 }}>{l}</span>
              <div style={{ display:"flex",gap:5,overflowX:"auto",WebkitOverflowScrolling:"touch",paddingBottom:2 }}>
                {items.map(it=><Pill key={it} active={s===it} onClick={()=>fn(it)}>{it}</Pill>)}
              </div>
            </div>
          ))}
        </div>

        {/* COUNT */}
        <div style={{ display:"flex",justifyContent:"space-between",paddingBottom:12,borderBottom:`1px solid ${C.border}`,marginBottom:16,flexWrap:"wrap",gap:4 }}>
          <span style={{ ...mono,fontSize:9,color:C.dim }}>{visF.length+lockF.length} RESULTS · {visF.length} ACCESSIBLE</span>
          {!mob&&<span style={{ ...mono,fontSize:8,color:C.vdim }}>UNDERGROUND 0 = MAINSTREAM · 100 = DEEP</span>}
        </div>

        {/* EVENTS */}
        {view==="grid" ? (
          <>
            {featured.length>0&&(
              <div style={{ display:"grid",gridTemplateColumns:featCols,gap:8,marginBottom:8 }}>
                {featured.map(e=><FeatCard key={e.id} e={e} onClick={setSel} mob={mob}/>)}
              </div>
            )}
            <div style={{ display:"grid",gridTemplateColumns:gridCols,gap:8 }}>
              {rest.map(e=><SmCard key={e.id} e={e} onClick={setSel}/>)}
              {!isPaid&&lockF.map(e=><LockedCard key={e.id} e={e} onUnlock={()=>setUpsell(true)}/>)}
            </div>
          </>
        ) : (
          <div style={{ display:"flex",flexDirection:"column",gap:2 }}>
            {visF.map(e=>mob?<SmCard key={e.id} e={e} onClick={setSel}/>:<ListRow key={e.id} e={e} onClick={setSel}/>)}
            {!isPaid&&lockF.map(e=><LockedCard key={e.id} e={e} onUnlock={()=>setUpsell(true)}/>)}
          </div>
        )}

        {/* FOOTER */}
        <div style={{ marginTop:60,paddingTop:18,borderTop:`1px solid ${C.border}`,display:"flex",justifyContent:"space-between",alignItems:"center",flexWrap:"wrap",gap:10 }}>
          <div style={{ ...display,fontSize:20,color:C.vdim,letterSpacing:5,fontWeight:300 }}>BASSLINE</div>
          {!mob&&<div style={{ display:"flex",gap:16 }}>{["DICE","RA","DJ MAG","BOILER ROOM"].map(s=><span key={s} style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2 }}>{s}</span>)}</div>}
          <div style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2 }}>THE MUSIC NEVER STOPS</div>
        </div>
      </div>

      {sel&&<EventModal e={sel} onClose={()=>setSel(null)} mob={mob}/>}
      {upsell&&<UpsellModal onClose={()=>setUpsell(false)} mob={mob}/>}
    </>
  );
}

/* ─── EVENT CARDS ────────────────────────────────────────────────────────── */
function FeatCard({ e, onClick, mob }) {
  const [hov,setHov]=useState(false);
  return (
    <div onClick={()=>onClick(e)} onMouseEnter={()=>setHov(true)} onMouseLeave={()=>setHov(false)}
      style={{ position:"relative",borderRadius:"10px",overflow:"hidden",cursor:"pointer",height:mob?340:400,border:`1px solid ${hov?C.gold+"40":C.border}`,background:"rgba(10,8,6,0.65)",backdropFilter:"blur(12px)",transition:"border-color 0.3s" }}>
      <div style={{ position:"absolute",inset:0,background:`radial-gradient(ellipse at bottom left,${C.gold}07 0%,transparent 60%)` }}/>
      <div style={{ position:"relative",zIndex:1,padding:mob?"20px":"28px 30px",height:"100%",display:"flex",flexDirection:"column",justifyContent:"space-between" }}>
        <div style={{ display:"flex",justifyContent:"space-between" }}>
          <div style={{ display:"flex",gap:7,flexWrap:"wrap" }}>
            <Pill active color={C.gold} sm>{e.tag}</Pill>
            {e.sold_out&&<Pill active color={C.err} sm>SOLD OUT</Pill>}
          </div>
          <span style={{ ...mono,fontSize:7,color:C.dim }}>{e.region}</span>
        </div>
        <div>
          <div style={{ ...display,fontSize:mob?"clamp(26px,6vw,42px)":"clamp(28px,4vw,52px)",color:C.ivory,lineHeight:1,fontWeight:600,marginBottom:5 }}>{e.name}</div>
          <div style={{ ...mono,fontSize:8,color:C.dim,marginBottom:12 }}>{e.sub}</div>
          {!mob&&<p style={{ ...mono,fontSize:10,color:C.stone,lineHeight:1.8,maxWidth:460,marginBottom:16 }}>{e.desc}</p>}
          <div style={{ display:"flex",gap:mob?14:22,flexWrap:"wrap",marginBottom:12 }}>
            {[["CITY",e.city],["DATES",e.date],!mob&&["TICKETS",e.ticket]].filter(Boolean).map(([l,v])=>(
              <div key={l}><div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2,marginBottom:2 }}>{l}</div><div style={{ ...mono,fontSize:10,color:C.stone }}>{v}</div></div>
            ))}
          </div>
          <div style={{ marginBottom:12 }}>
            <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2,marginBottom:5 }}>UNDERGROUND RATING</div>
            <Ruler val={e.u}/>
          </div>
          {!mob&&<div style={{ display:"flex",gap:5,flexWrap:"wrap" }}>
            {e.artists.slice(0,3).map(a=><span key={a} style={{ ...mono,fontSize:8,color:C.stone,padding:"3px 8px",border:`1px solid ${C.border}`,borderRadius:"100px" }}>{a}</span>)}
            {e.artists.length>3&&<span style={{ ...mono,fontSize:8,color:C.dim }}>+{e.artists.length-3}</span>}
          </div>}
        </div>
      </div>
    </div>
  );
}

function SmCard({ e, onClick }) {
  const [hov,setHov]=useState(false);
  return (
    <div onClick={()=>onClick(e)} onMouseEnter={()=>setHov(true)} onMouseLeave={()=>setHov(false)}
      style={{ background:"rgba(12,10,8,0.75)",backdropFilter:"blur(12px)",border:`1px solid ${hov?C.gold+"35":C.border}`,borderRadius:"10px",cursor:"pointer",display:"flex",flexDirection:"column",transition:"all 0.2s",transform:hov?"translateY(-2px)":"none" }}>
      <div style={{ padding:"15px 16px 0" }}>
        <div style={{ display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:10 }}>
          <Pill active color={C.gold} sm>{e.tag}</Pill>
          <div style={{ display:"flex",gap:5,alignItems:"center" }}>
            {e.sold_out&&<Pill active color={C.err} sm>SOLD OUT</Pill>}
            <Dot size={6}/>
          </div>
        </div>
        <div style={{ ...display,fontSize:21,color:C.ivory,fontWeight:600,lineHeight:1.1 }}>{e.name}</div>
        <div style={{ ...mono,fontSize:8,color:C.dim,marginTop:2,marginBottom:10 }}>{e.sub}</div>
      </div>
      <div style={{ height:1,background:C.border,margin:"0 16px" }}/>
      <div style={{ padding:"10px 16px 16px",display:"flex",flexDirection:"column",gap:9 }}>
        <div style={{ display:"flex",gap:16 }}>
          <div><div style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2 }}>CITY</div><div style={{ ...mono,fontSize:10,color:C.stone }}>{e.city}</div></div>
          <div><div style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2 }}>VIA</div><div style={{ ...mono,fontSize:10,color:C.gold }}>{e.ticket}</div></div>
        </div>
        <div style={{ display:"flex",gap:5,flexWrap:"wrap" }}>
          {e.genre.slice(0,2).map(g=><Pill key={g} sm>{g}</Pill>)}
        </div>
        <div>
          <div style={{ ...mono,fontSize:7,color:C.vdim,letterSpacing:2,marginBottom:4 }}>UNDERGROUND</div>
          <Ruler val={e.u}/>
        </div>
      </div>
    </div>
  );
}

function LockedCard({ e, onUnlock }) {
  return (
    <div onClick={onUnlock} style={{ background:"rgba(10,8,6,0.5)",border:`1px solid ${C.border}`,borderRadius:"10px",cursor:"pointer",minHeight:180,position:"relative",overflow:"hidden" }}>
      <div style={{ position:"absolute",inset:0,backdropFilter:"blur(6px)",background:"rgba(8,7,5,0.8)",zIndex:2,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",gap:10 }}>
        <svg width="18" height="18" viewBox="0 0 18 18" fill="none"><rect x="2" y="7" width="14" height="10" rx="1" stroke={C.stone} strokeWidth="1.2"/><path d="M5 7V5a4 4 0 018 0v2" stroke={C.stone} strokeWidth="1.2"/></svg>
        <div style={{ ...mono,fontSize:8,color:C.stone,letterSpacing:2 }}>MEMBER ACCESS</div>
        <div style={{ background:`${C.gold}18`,border:`1px solid ${C.gold}40`,borderRadius:"100px",padding:"5px 14px",...mono,fontSize:8,color:C.gold,letterSpacing:2 }}>UNLOCK →</div>
      </div>
      <div style={{ padding:"16px",opacity:0.1 }}>
        <div style={{ ...display,fontSize:18,color:C.ivory,fontWeight:600 }}>{e.name}</div>
        <div style={{ ...mono,fontSize:8,color:C.dim,marginTop:3 }}>{e.city}</div>
      </div>
    </div>
  );
}

function ListRow({ e, onClick }) {
  const [hov,setHov]=useState(false);
  return (
    <div onClick={()=>onClick(e)} onMouseEnter={()=>setHov(true)} onMouseLeave={()=>setHov(false)}
      style={{ background:hov?"rgba(14,12,9,0.85)":"rgba(10,8,6,0.65)",backdropFilter:"blur(12px)",border:`1px solid ${hov?C.gold+"28":C.border}`,borderLeft:`2px solid ${hov?C.gold:C.border}`,borderRadius:"4px",padding:"12px 16px",cursor:"pointer",display:"grid",gridTemplateColumns:"18px 1fr 100px 110px 80px 90px 50px",gap:14,alignItems:"center",transition:"all 0.15s" }}>
      <Dot size={6}/>
      <div><div style={{ ...display,fontSize:14,color:C.ivory,fontWeight:600 }}>{e.name}</div><div style={{ ...mono,fontSize:8,color:C.dim }}>{e.sub}</div></div>
      <div style={{ ...mono,fontSize:9,color:C.stone }}>{e.city}</div>
      <div style={{ ...mono,fontSize:8,color:C.dim }}>{e.genre[0]}</div>
      <div style={{ ...mono,fontSize:9,color:C.gold }}>{e.ticket}</div>
      <Ruler val={e.u}/>
      <Pill active sm color={C.gold}>{e.tag}</Pill>
    </div>
  );
}

/* ─── EVENT MODAL ────────────────────────────────────────────────────────── */
function EventModal({ e, onClose, mob }) {
  return (
    <div onClick={onClose} style={{ position:"fixed",inset:0,zIndex:200,background:"rgba(4,3,6,0.94)",backdropFilter:"blur(16px)",display:"flex",alignItems:mob?"flex-end":"center",justifyContent:"center",padding:mob?0:20,animation:"fadeIn 0.18s ease" }}>
      <div onClick={ev=>ev.stopPropagation()} style={{ background:"rgba(12,10,8,0.97)",backdropFilter:"blur(20px)",border:`1px solid ${C.border}`,borderRadius:mob?"12px 12px 0 0":"10px",width:"100%",maxWidth:mob?"100%":680,maxHeight:mob?"90vh":"88vh",overflow:"auto",animation:mob?"slideIn 0.28s ease":"slideUp 0.22s ease" }}>
        {/* Drag handle on mobile */}
        {mob&&<div style={{ display:"flex",justifyContent:"center",padding:"10px 0 0" }}><div style={{ width:36,height:3,borderRadius:2,background:C.border }}/></div>}
        <div style={{ padding:mob?"16px 20px":"24px 28px",borderBottom:`1px solid ${C.border}`,display:"flex",justifyContent:"space-between",alignItems:"flex-start" }}>
          <div>
            <div style={{ display:"flex",gap:7,marginBottom:8,flexWrap:"wrap" }}>
              <Pill active color={C.gold} sm>{e.tag}</Pill>
              {e.sold_out&&<Pill active color={C.err} sm>SOLD OUT</Pill>}
            </div>
            <div style={{ ...display,fontSize:mob?32:40,color:C.ivory,lineHeight:1,fontWeight:600 }}>{e.name}</div>
            <div style={{ ...mono,fontSize:9,color:C.dim,marginTop:4 }}>{e.sub}</div>
          </div>
          {!mob&&<button onClick={onClose} style={{ background:"rgba(20,16,12,0.8)",border:`1px solid ${C.border}`,color:C.stone,width:30,height:30,borderRadius:"50%",fontSize:14,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0,marginTop:2 }}>×</button>}
        </div>
        <div style={{ padding:mob?"16px 20px 24px":"22px 28px" }}>
          <p style={{ ...mono,fontSize:mob?12:11,color:C.stone,lineHeight:1.9,marginBottom:22 }}>{e.desc}</p>
          <div style={{ display:"grid",gridTemplateColumns:mob?"1fr 1fr":"1fr 1fr 1fr",gap:14,marginBottom:20 }}>
            {[["CITY",`${e.city}, ${e.country}`],["VENUE",e.venue],["DATES",e.date],["CAPACITY",e.cap],["TICKETS",e.ticket],["REGION",e.region]].map(([l,v])=>(
              <div key={l}><div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2.5,marginBottom:4 }}>{l}</div><div style={{ ...mono,fontSize:mob?11:10,color:C.stone }}>{v}</div></div>
            ))}
          </div>
          <div style={{ marginBottom:18 }}>
            <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2.5,marginBottom:6 }}>UNDERGROUND RATING — {e.u}/100</div>
            <Ruler val={e.u}/>
          </div>
          <div style={{ marginBottom:16 }}>
            <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2.5,marginBottom:7 }}>GENRES</div>
            <div style={{ display:"flex",gap:6,flexWrap:"wrap" }}>{e.genre.map(g=><Pill key={g} sm>{g}</Pill>)}</div>
          </div>
          <div>
            <div style={{ ...mono,fontSize:7,color:C.dim,letterSpacing:2.5,marginBottom:7 }}>ARTISTS</div>
            <div style={{ display:"flex",gap:6,flexWrap:"wrap" }}>
              {e.artists.map(a=><span key={a} style={{ ...mono,fontSize:mob?11:10,color:C.stone,padding:"5px 10px",border:`1px solid ${C.border}`,borderRadius:"100px" }}>{a}</span>)}
            </div>
          </div>
          {mob&&<button onClick={onClose} style={{ display:"block",width:"100%",marginTop:20,padding:"13px",background:"transparent",border:`1px solid ${C.border}`,borderRadius:"8px",...mono,fontSize:10,color:C.stone,letterSpacing:2 }}>CLOSE</button>}
        </div>
      </div>
    </div>
  );
}

/* ─── UPSELL MODAL ───────────────────────────────────────────────────────── */
function UpsellModal({ onClose, mob }) {
  const [step, setStep]   = useState("plans");
  const [chosen, setChosen] = useState(null);

  return (
    <div onClick={onClose} style={{ position:"fixed",inset:0,zIndex:300,background:"rgba(4,3,6,0.95)",backdropFilter:"blur(18px)",display:"flex",alignItems:mob?"flex-end":"center",justifyContent:"center",padding:mob?0:20,animation:"fadeIn 0.2s ease" }}>
      <div onClick={ev=>ev.stopPropagation()} style={{ background:"rgba(12,10,8,0.97)",backdropFilter:"blur(20px)",border:`1px solid ${C.border}`,borderRadius:mob?"12px 12px 0 0":"10px",width:"100%",maxWidth:mob?"100%":640,maxHeight:mob?"90vh":"90vh",overflow:"auto",animation:mob?"slideIn 0.28s ease":"slideUp 0.24s ease" }}>
        {mob&&<div style={{ display:"flex",justifyContent:"center",padding:"10px 0 0" }}><div style={{ width:36,height:3,borderRadius:2,background:C.border }}/></div>}
        <div style={{ padding:"18px 20px",borderBottom:`1px solid ${C.border}`,display:"flex",justifyContent:"space-between",alignItems:"center" }}>
          <div>
            {step==="checkout"&&<button onClick={()=>setStep("plans")} style={{ background:"none",border:"none",...mono,fontSize:8,color:C.stone,letterSpacing:2,display:"block",marginBottom:4 }}>← BACK</button>}
            <div style={{ ...display,fontSize:22,color:C.ivory,fontWeight:600 }}>{step==="plans"?"Unlock Full Access":`Checkout · ${chosen?.name}`}</div>
            <div style={{ ...mono,fontSize:8,color:C.dim,letterSpacing:2,marginTop:2 }}>{step==="plans"?`${ALL_EVENTS.length-GUEST_LIMIT} more events behind this door`:`${chosen?.price}${chosen?.freq}`}</div>
          </div>
          <button onClick={onClose} style={{ background:"rgba(20,16,12,0.8)",border:`1px solid ${C.border}`,color:C.stone,width:28,height:28,borderRadius:"50%",fontSize:14,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0 }}>×</button>
        </div>

        {step==="plans"&&(
          <div style={{ padding:"18px 20px" }}>
            <div style={{ display:"flex",flexDirection:"column",gap:10,marginBottom:16 }}>
              {TIERS.filter(t=>t.id!=="free").map(tier=>(
                <div key={tier.id} style={{ background:"rgba(14,12,9,0.8)",border:`1px solid ${tier.id==="founding"?C.gold+"30":C.border}`,borderRadius:"8px",padding:"18px 16px",position:"relative",overflow:"hidden" }}>
                  {tier.badge&&<div style={{ position:"absolute",top:0,right:0,background:C.gold,...mono,fontSize:7,color:"#080806",padding:"4px 10px",letterSpacing:2 }}>{tier.badge}</div>}
                  <div style={{ ...display,fontSize:38,color:tier.color,lineHeight:1,fontWeight:300 }}>{tier.price}<span style={{ ...mono,fontSize:12,color:C.stone }}>{tier.freq}</span></div>
                  <div style={{ ...display,fontSize:20,color:C.ivory,letterSpacing:3,fontWeight:600,marginBottom:5 }}>{tier.name}</div>
                  <div style={{ ...mono,fontSize:9,color:C.stone,lineHeight:1.7,marginBottom:12 }}>{tier.desc}</div>
                  {tier.features.slice(0,3).map(f=>(
                    <div key={f} style={{ display:"flex",gap:8,marginBottom:5 }}><span style={{ color:tier.color,fontSize:9,marginTop:1 }}>✓</span><span style={{ ...mono,fontSize:9,color:C.stone }}>{f}</span></div>
                  ))}
                  <button onClick={()=>{ setChosen(tier); setStep("checkout"); }}
                    style={{ width:"100%",marginTop:14,padding:"12px",borderRadius:"6px",...display,fontSize:17,fontWeight:600,letterSpacing:3,background:tier.id==="founding"?`linear-gradient(135deg,${C.goldHi},${C.gold})`:`linear-gradient(135deg,${C.gold},${C.goldDim}CC)`,color:"#0A0806",transition:"opacity 0.15s" }}
                    onMouseEnter={e=>e.currentTarget.style.opacity="0.8"} onMouseLeave={e=>e.currentTarget.style.opacity="1"}>
                    {tier.cta} →
                  </button>
                </div>
              ))}
            </div>
            <div style={{ display:"flex",gap:20,justifyContent:"center",flexWrap:"wrap" }}>
              {["🔒 Secure","↩ Cancel anytime","⚡ Instant"].map(t=><span key={t} style={{ ...mono,fontSize:8,color:C.dim }}>{t}</span>)}
            </div>
          </div>
        )}

        {step==="checkout"&&chosen&&(
          <div style={{ padding:"18px 20px" }}>
            <div style={{ background:"rgba(14,12,9,0.8)",border:`1px solid ${chosen.id==="founding"?C.gold+"30":C.border}`,borderRadius:"8px",padding:"16px",marginBottom:18 }}>
              <div style={{ ...display,fontSize:36,color:chosen.id==="founding"?C.goldHi:C.gold,lineHeight:1,fontWeight:300 }}>{chosen.price}<span style={{ ...mono,fontSize:12,color:C.stone }}>{chosen.freq}</span></div>
              <div style={{ ...display,fontSize:20,color:C.ivory,letterSpacing:3,fontWeight:600,marginTop:3,marginBottom:5 }}>{chosen.name}</div>
              {chosen.id==="founding"&&<div style={{ ...mono,fontSize:8,color:C.ok }}>SAVE $65 vs monthly billing</div>}
              <div style={{ display:"flex",justifyContent:"space-between",alignItems:"center",borderTop:`1px solid ${C.border}`,marginTop:12,paddingTop:10 }}>
                <span style={{ ...mono,fontSize:9,color:C.ivory }}>TOTAL TODAY</span>
                <span style={{ ...display,fontSize:22,color:chosen.id==="founding"?C.goldHi:C.gold,fontWeight:300 }}>{chosen.price}</span>
              </div>
            </div>
            <PaymentForm tier={chosen} onSuccess={onClose}/>
          </div>
        )}
      </div>
    </div>
  );
}

/* ─── MUSIC PLAYER ───────────────────────────────────────────────────────── */
function MusicPlayer() {
  const [ready,    setReady]    = useState(false);
  const [playing,  setPlaying]  = useState(false);
  const [muted,    setMuted]    = useState(false);
  const [volume,   setVolume]   = useState(72);
  const [trackIdx, setTrackIdx] = useState(0);
  const [tick,     setTick]     = useState(0);
  const [expanded, setExpanded] = useState(false);
  const playerRef  = useRef(null);
  const divRef     = useRef(null);
  const apiLoaded  = useRef(false);
  const w = useW();
  const mob = w < 640;

  useEffect(()=>{ const id=setInterval(()=>setTick(t=>t+1),90); return()=>clearInterval(id); },[]);

  useEffect(() => {
    if(apiLoaded.current) return;
    apiLoaded.current = true;
    const create = () => {
      if(!divRef.current||!window.YT?.Player) return;
      playerRef.current = new window.YT.Player(divRef.current, {
        height:"1", width:"1", videoId:TRACKS[0].id,
        playerVars:{ autoplay:0,controls:0,disablekb:1,rel:0,modestbranding:1 },
        events:{
          onReady:(e)=>{ e.target.setVolume(72); setReady(true); },
          onStateChange:(e)=>{
            const S=window.YT?.PlayerState; if(!S) return;
            if(e.data===S.PLAYING) setPlaying(true);
            if(e.data===S.PAUSED||e.data===S.CUED) setPlaying(p=>e.data===S.PAUSED?false:p);
            if(e.data===S.ENDED) skip(1);
          },
          onError:()=>skip(1),
        },
      });
    };
    if(window.YT?.Player){ create(); }
    else {
      const s=document.createElement("script"); s.src="https://www.youtube.com/iframe_api"; s.async=true; document.head.appendChild(s);
      const prev=window.onYouTubeIframeAPIReady;
      window.onYouTubeIframeAPIReady=()=>{ prev?.(); create(); };
    }
    return()=>{ try{ playerRef.current?.destroy?.(); }catch(_){} };
  },[]);

  const skip = (dir) => {
    setTrackIdx(i=>{
      const next=(i+dir+TRACKS.length)%TRACKS.length;
      try{ playerRef.current?.loadVideoById?.(TRACKS[next].id); }catch(_){}
      return next;
    });
  };

  const togglePlay = () => {
    if(!ready) return;
    try{ playing?playerRef.current?.pauseVideo?.():playerRef.current?.playVideo?.(); }catch(_){}
  };

  const handleVol = (v) => {
    setVolume(v);
    if(muted){ setMuted(false); playerRef.current?.unMute?.(); }
    try{ playerRef.current?.setVolume?.(v); }catch(_){}
  };

  const toggleMute = () => {
    setMuted(m=>{ try{ m?playerRef.current?.unMute?.():playerRef.current?.mute?.(); }catch(_){} return !m; });
  };

  const track  = TRACKS[trackIdx];
  const volPct = muted?0:volume;
  const bars   = Array.from({length:mob?14:22},(_,i)=>playing?Math.max(3,6+Math.abs(Math.sin((tick+i*3.7)*0.28))*20):3);

  return (
    <>
      <div ref={divRef} style={{ position:"fixed",left:"-9999px",top:"-9999px",width:1,height:1,pointerEvents:"none",zIndex:-1 }}/>

      <div style={{ position:"fixed",bottom:0,left:0,right:0,zIndex:500,background:"rgba(8,6,4,0.97)",backdropFilter:"blur(28px)",borderTop:`1px solid ${C.border}` }}>

        {/* Mobile expanded view */}
        {mob&&expanded&&(
          <div style={{ padding:"20px 20px 4px",borderBottom:`1px solid ${C.border}`,animation:"slideUp 0.2s ease" }}>
            <div style={{ ...display,fontSize:22,color:C.ivory,fontWeight:600,marginBottom:3 }}>{track.title}</div>
            <div style={{ ...mono,fontSize:9,color:C.stone,marginBottom:2 }}>{track.artist}</div>
            <div style={{ ...mono,fontSize:8,color:C.goldDim,letterSpacing:1.5,marginBottom:14 }}>{track.label}</div>
            {/* Volume on expanded mobile */}
            <div style={{ display:"flex",alignItems:"center",gap:10,marginBottom:12 }}>
              <button onClick={toggleMute} style={{ background:"none",border:"none",color:muted?C.dim:C.stone,display:"flex" }}>
                {muted?<svg width="15" height="13" viewBox="0 0 15 13" fill="none"><path d="M1 4.5h2.5L7 1v11L3.5 8.5H1V4.5z" stroke="currentColor" strokeWidth="1.2"/><line x1="10" y1="4.5" x2="14" y2="8.5" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/><line x1="14" y1="4.5" x2="10" y2="8.5" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/></svg>:<svg width="15" height="13" viewBox="0 0 15 13" fill="none"><path d="M1 4.5h2.5L7 1v11L3.5 8.5H1V4.5z" stroke="currentColor" strokeWidth="1.2"/><path d="M9.5 4c.9.8 1.5 1.8 1.5 2.5S10.4 8.3 9.5 9" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/></svg>}
              </button>
              <input type="range" min="0" max="100" value={volPct} onChange={e=>handleVol(Number(e.target.value))}
                style={{ flex:1,height:2,background:`linear-gradient(to right,${C.gold} 0%,${C.gold} ${volPct}%,${C.border} ${volPct}%,${C.border} 100%)`,borderRadius:1 }}/>
            </div>
          </div>
        )}

        {/* Main bar */}
        <div style={{ display:"flex",alignItems:"center",padding:"0 16px",height:56,gap:mob?12:20 }}>
          {/* Waveform */}
          <div onClick={mob?()=>setExpanded(p=>!p):undefined} style={{ display:"flex",alignItems:"center",gap:1.5,height:26,flexShrink:0,cursor:mob?"pointer":undefined }}>
            {bars.map((h,i)=>(
              <div key={i} style={{ width:2,borderRadius:1,background:playing?`linear-gradient(180deg,${C.goldHi},${C.goldDim})`:C.dim,height:`${h}px`,transition:playing?"height 0.09s ease":"height 0.4s ease,background 0.4s",opacity:playing?0.55+(i%4)*0.12:0.25 }}/>
            ))}
          </div>

          {/* Track info — hidden on mobile when collapsed */}
          {(!mob||(mob&&!expanded))&&(
            <div style={{ flex:1,minWidth:0,display:"flex",alignItems:"center",gap:10 }}>
              <div style={{ minWidth:0 }} onClick={mob?()=>setExpanded(p=>!p):undefined} >
                <div style={{ ...display,fontSize:mob?14:15,color:C.ivory,fontWeight:600,letterSpacing:0.5,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis",cursor:mob?"pointer":undefined }}>
                  {track.title}
                </div>
                {!mob&&<div style={{ display:"flex",gap:7,alignItems:"center",marginTop:1 }}>
                  <span style={{ ...mono,fontSize:8,color:C.stone }}>{track.artist}</span>
                  <span style={{ ...mono,fontSize:7,color:C.dim }}>·</span>
                  <span style={{ ...mono,fontSize:7,color:C.goldDim }}>{track.label}</span>
                </div>}
              </div>
              {playing&&(
                <div style={{ display:"flex",alignItems:"center",gap:5,background:`${C.gold}10`,border:`1px solid ${C.gold}25`,borderRadius:"100px",padding:"2px 8px",flexShrink:0 }}>
                  <div style={{ width:5,height:5,borderRadius:"50%",background:C.gold,animation:"pulse 1.5s infinite" }}/>
                  <span style={{ ...mono,fontSize:7,color:C.gold,letterSpacing:2 }}>LIVE</span>
                </div>
              )}
            </div>
          )}
          {mob&&expanded&&<div style={{ flex:1 }}/>}

          {/* Controls */}
          <div style={{ display:"flex",alignItems:"center",gap:mob?14:16,flexShrink:0 }}>
            <button onClick={()=>skip(-1)} style={{ background:"none",border:"none",color:C.stone,width:28,height:28,display:"flex",alignItems:"center",justifyContent:"center",transition:"color 0.2s" }}
              onMouseEnter={e=>e.currentTarget.style.color=C.gold} onMouseLeave={e=>e.currentTarget.style.color=C.stone}>
              <svg width="14" height="14" viewBox="0 0 14 14" fill="none"><path d="M10 2.5L5 7l5 4.5" stroke="currentColor" strokeWidth="1.3" strokeLinecap="round" strokeLinejoin="round"/><line x1="2.5" y1="2.5" x2="2.5" y2="11.5" stroke="currentColor" strokeWidth="1.3" strokeLinecap="round"/></svg>
            </button>
            <button onClick={togglePlay} style={{ width:38,height:38,borderRadius:"50%",background:ready?`${C.gold}1A`:`${C.border}40`,border:`1px solid ${ready?C.gold+"45":C.border}`,color:ready?C.gold:C.dim,display:"flex",alignItems:"center",justifyContent:"center",transition:"all 0.22s" }}>
              {!ready?<span style={{ width:10,height:10,border:`1.5px solid ${C.dim}`,borderTop:`1.5px solid ${C.stone}`,borderRadius:"50%",display:"inline-block",animation:"spin 1s linear infinite" }}/>
                :playing?<svg width="12" height="14" viewBox="0 0 12 14" fill="none"><rect x="0" y="0" width="4.5" height="14" rx="1" fill="currentColor"/><rect x="7.5" y="0" width="4.5" height="14" rx="1" fill="currentColor"/></svg>
                :<svg width="13" height="15" viewBox="0 0 13 15" fill="none"><path d="M1.5 1l10 6.5-10 6.5V1z" fill="currentColor"/></svg>}
            </button>
            <button onClick={()=>skip(1)} style={{ background:"none",border:"none",color:C.stone,width:28,height:28,display:"flex",alignItems:"center",justifyContent:"center",transition:"color 0.2s" }}
              onMouseEnter={e=>e.currentTarget.style.color=C.gold} onMouseLeave={e=>e.currentTarget.style.color=C.stone}>
              <svg width="14" height="14" viewBox="0 0 14 14" fill="none"><path d="M4 2.5l5 4.5-5 4.5" stroke="currentColor" strokeWidth="1.3" strokeLinecap="round" strokeLinejoin="round"/><line x1="11.5" y1="2.5" x2="11.5" y2="11.5" stroke="currentColor" strokeWidth="1.3" strokeLinecap="round"/></svg>
            </button>
          </div>

          {/* Volume — desktop only */}
          {!mob&&(
            <div style={{ display:"flex",alignItems:"center",gap:10,flexShrink:0 }}>
              <span style={{ ...mono,fontSize:8,color:C.dim }}>{String(trackIdx+1).padStart(2,"0")}/{String(TRACKS.length).padStart(2,"0")}</span>
              <button onClick={toggleMute} style={{ background:"none",border:"none",color:muted?C.dim:C.stone,display:"flex",alignItems:"center",transition:"color 0.2s" }}
                onMouseEnter={e=>e.currentTarget.style.color=C.gold} onMouseLeave={e=>e.currentTarget.style.color=muted?C.dim:C.stone}>
                {muted?<svg width="16" height="13" viewBox="0 0 16 13" fill="none"><path d="M1 4.5h2.5L8 1v11L3.5 8.5H1V4.5z" stroke="currentColor" strokeWidth="1.2"/><line x1="11" y1="4.5" x2="15" y2="8.5" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/><line x1="15" y1="4.5" x2="11" y2="8.5" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/></svg>:<svg width="16" height="13" viewBox="0 0 16 13" fill="none"><path d="M1 4.5h2.5L8 1v11L3.5 8.5H1V4.5z" stroke="currentColor" strokeWidth="1.2"/><path d="M10 4c1 .9 1.6 1.9 1.6 2.75S11 8.1 10 9" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/><path d="M12.5 2.5c1.8 1.4 2.8 3 2.8 4.75S14.3 9.8 12.5 11.2" stroke="currentColor" strokeWidth="1.2" strokeLinecap="round"/></svg>}
              </button>
              <input type="range" min="0" max="100" value={volPct} onChange={e=>handleVol(Number(e.target.value))}
                style={{ width:80,height:2,background:`linear-gradient(to right,${C.gold} 0%,${C.gold} ${volPct}%,${C.border} ${volPct}%,${C.border} 100%)`,borderRadius:1 }}/>
            </div>
          )}
        </div>
      </div>
    </>
  );
}

/* ─── ROOT ───────────────────────────────────────────────────────────────── */
export default function App() {
  const [phase, setPhase] = useState("gate");
  const [tier,  setTier]  = useState(null);
  return (
    <>
      <style>{CSS}</style>
      {phase==="gate"    && <PasswordGate onSuccess={()=>setPhase("pricing")}/>}
      {phase==="pricing" && <PricingScreen onSelect={t=>{ setTier(t); setPhase("app"); }}/>}
      {phase==="app"     && <MainApp tier={tier} onLogout={()=>{ setPhase("gate"); setTier(null); }}/>}
      <MusicPlayer />
    </>
  );
}
