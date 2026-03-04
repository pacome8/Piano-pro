[piano-pro-v2 (3).html](https://github.com/user-attachments/files/25737883/piano-pro-v2.3.html)
<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8"/>
<meta name="viewport" content="width=device-width,initial-scale=1.0,viewport-fit=cover"/>
<title>🎹 Pacôme — Piano Pro</title>
<meta name="mobile-web-app-capable" content="yes"/>
<meta name="apple-mobile-web-app-capable" content="yes"/>
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>
<meta name="apple-mobile-web-app-title" content="Piano Pro"/>
<meta name="theme-color" content="#0f0e17"/>
<link rel="manifest" href="manifest.json"/>
<!-- Apple touch icons (inline data URI pour fichier unique) -->
<link rel="apple-touch-icon" href="data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 192 192'%3E%3Crect width='192' height='192' rx='40' fill='%230f0e17'/%3E%3Ctext x='96' y='130' font-size='100' text-anchor='middle'%3E🎹%3C/text%3E%3C/svg%3E"/>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
<link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500;600;700&display=swap" rel="stylesheet"/>
<style>
*{box-sizing:border-box;margin:0;padding:0;}
html,body{height:100%;font-family:"DM Sans",sans-serif;}
::-webkit-scrollbar{display:none;}
@keyframes slideUp{from{transform:translateY(100%)}to{transform:translateY(0)}}
@keyframes pulseR{0%,100%{opacity:1}50%{opacity:.3}}
@keyframes goldGlow{0%,100%{box-shadow:0 0 8px rgba(251,191,36,0.4)}50%{box-shadow:0 0 22px rgba(251,191,36,0.7)}}
</style>
</head>
<body>
<div id="root"></div>
<script type="text/babel">
const {useState,useMemo,useEffect} = React;

const MFR=["Janvier","Février","Mars","Avril","Mai","Juin","Juillet","Août","Septembre","Octobre","Novembre","Décembre"];
const DFR=["Dim","Lun","Mar","Mer","Jeu","Ven","Sam"];
const DFULL=["Dimanche","Lundi","Mardi","Mercredi","Jeudi","Vendredi","Samedi"];
const PMODES=["Virement","CESU","CESU+","Espèces","Chèque","Envoyé non reçu","Autre"];
let _ID=100,_RID=20;

const TC={
  lesson:{label:"Cours",color:"#3b82f6",icon:"🎵"},      // bleu vif
  gig:{label:"Prestation",color:"#f97316",icon:"🎤"},     // orange vif
  personal:{label:"Personnel",color:"#ec4899",icon:"📖"}, // rose/fuchsia
  work:{label:"CDI",color:"#10b981",icon:"💼"},           // vert émeraude
  task:{label:"Tâche",color:"#f59e0b",icon:"📌"},         // ambre/jaune
};

const SC={
  "Prévu":{bg:"rgba(148,163,184,0.12)",text:"#94a3b8",b:"rgba(148,163,184,0.25)",ico:"🗺️"},
  "Fait":{bg:"rgba(52,211,153,0.12)",text:"#34d399",b:"rgba(52,211,153,0.25)",ico:"✅"},
  "À Rattraper":{bg:"rgba(251,191,36,0.12)",text:"#fbbf24",b:"rgba(251,191,36,0.25)",ico:"🔄"},
  "Annulé":{bg:"rgba(248,113,113,0.08)",text:"#6b7280",b:"rgba(248,113,113,0.15)",ico:"❌"},
  "Vacances":{bg:"rgba(156,163,175,0.08)",text:"#6b7280",b:"rgba(156,163,175,0.15)",ico:"🏖️"},
};

const questStatus = ev => {
  if(ev.type==="task") return {key:"Task",ico:"📌",label:"À faire",col:"#94a3b8",bg:"rgba(148,163,184,0.08)",b:"rgba(148,163,184,0.2)"};
  if(ev.status==="Annulé") return {key:"Annulé",ico:"❌",label:"Annulé",col:"#6b7280",bg:"rgba(107,114,128,0.1)",b:"rgba(107,114,128,0.2)"};
  if(ev.status==="Vacances") return {key:"Vacances",ico:"🏖️",label:"Vacances",col:"#6b7280",bg:"rgba(107,114,128,0.1)",b:"rgba(107,114,128,0.2)"};
  if(ev.status==="À Rattraper") return {key:"Rattrapage",ico:"🔄",label:"Rattrapage",col:"#fbbf24",bg:"rgba(251,191,36,0.12)",b:"rgba(251,191,36,0.3)"};
  if(ev.status==="Prévu") return {key:"Aventure",ico:"🗺️",label:"Prochaine Aventure",col:"#94a3b8",bg:"rgba(148,163,184,0.1)",b:"rgba(148,163,184,0.22)"};
  if(ev.status==="Fait"){
    const days=Math.floor((Date.now()-new Date(ev.date+"T12:00"))/(864e5));
    if(ev.paid) return {key:"Accomplie",ico:"⚔️",label:"Quête Accomplie",col:"#fbbf24",bg:"rgba(251,191,36,0.15)",b:"rgba(251,191,36,0.35)"};
    if(days>14) return {key:"Alerte",ico:"🚨",label:"Alerte Butin",col:"#f87171",bg:"rgba(248,113,113,0.15)",b:"rgba(248,113,113,0.35)"};
    return {key:"EnRoute",ico:"📦",label:"Butin en Route",col:"#60a5fa",bg:"rgba(96,165,250,0.12)",b:"rgba(96,165,250,0.3)"};
  }
  return {key:"Aventure",ico:"🗺️",label:"Prochaine Aventure",col:"#94a3b8",bg:"rgba(148,163,184,0.1)",b:"rgba(148,163,184,0.22)"};
};

const fd=ds=>{if(!ds)return"—";const d=new Date(ds+"T12:00");return DFR[d.getDay()]+" "+d.getDate()+" "+MFR[d.getMonth()].slice(0,3);};
const dAgo=ds=>Math.floor((Date.now()-new Date(ds+"T12:00"))/(864e5));

const COMPANY={name:"Schulte Marketing",siret:"923 753 438",iban:"FR76 1027 8361 7300 0138 8280 481",bic:"CMCIFR2A",artist:"Pacôme Schulte",phone:"07 69 13 96 99"};

const MAILS0=[
  {id:"ehpad",ico:"🏥",label:"EHPAD / Résidence",
   sub:"Partager la musique avec vos résidents — Pacôme Schulte",
   body:"Bonjour,\n\nJe m'appelle Pacôme Schulte, j'ai 22 ans, et je suis un jeune pianiste avec un parcours qui mélange l'apprentissage en autodidacte et le Conservatoire. Je vous contacte car j'ai à cœur de partager ma musique avec ceux qui en ont le plus besoin, notamment les personnes âgées ou en situation de handicap, pour qui l'accès à l'art est essentiel.\n\nJ'ai aujourd'hui une pratique très variée : je suis le pianiste officiel du FC Nantes, je joue pour des mariages et dans des restaurants, mais mes moments de jeu en EHPAD restent pour moi les plus riches en émotion.\n\nMa façon de jouer est très personnelle. J'aime laisser l'émotion du moment guider mes mains. Je propose un mélange de compositions originales, d'improvisations douces et de mélodies chaleureuses qui parlent à la mémoire et à l'âme.\n\nPour que vous puissiez voir concrètement l'effet de ma musique sur vos résidents, je serais ravi de venir vous offrir une première heure de jeu à titre gracieux, sans aucun engagement de votre part.\n\nSerait-il possible de convenir d'une date ?\n\n{BIO}\n\nBien cordialement,\nPacôme Schulte\n07 69 13 96 99"},
  {id:"wedding",ico:"💍",label:"Wedding Planner",
   sub:"Un pianiste pour sublimer vos mariages — Pacôme Schulte",
   body:"Bonjour,\n\nJe m'appelle Pacôme Schulte, j'ai 22 ans et je suis pianiste. Je me permets de vous contacter parce que j'admire votre travail de Wedding Planner et je pense que ma façon de jouer pourrait vraiment plaire à vos futurs mariés.\n\nJe ne suis pas dans le formatage habituel : je suis le pianiste officiel du FC Nantes, donc j'ai l'habitude de l'exigence, mais dans les mariages, je cherche surtout l'émotion pure et l'élégance.\n\nMon style est très axé sur l'improvisation et la création d'une atmosphère sur-mesure pour chaque couple. Je ne cherche pas juste à être un prestataire de plus, mais un vrai partenaire pour vous aider à sublimer vos événements.\n\n{BIO}\n\nAu plaisir d'échanger avec vous,\nPacôme Schulte\n07 69 13 96 99"},
  {id:"resto",ico:"🍽️",label:"Restaurant / Lounge",
   sub:"Un piano pour l'âme de votre établissement — Pacôme Schulte",
   body:"Bonjour,\n\nJe m'appelle Pacôme Schulte, j'ai 22 ans, et je suis pianiste (notamment pour le FC Nantes). Je passe souvent devant votre établissement et j'aime beaucoup l'âme du lieu. Je suis convaincu que mon piano pourrait s'y intégrer parfaitement.\n\nMon style, c'est de l'improvisation douce et mélodieuse, rien de rigide ou de pré-enregistré. L'idée, c'est vraiment de créer une bulle où les clients se posent et se sentent bien dès qu'ils entrent.\n\nJe peux passer vous jouer quelques morceaux quand vous voulez, juste pour le feeling, sans engagement.\n\n{BIO}\n\nÀ bientôt j'espère,\nPacôme Schulte\n07 69 13 96 99"},
];

const STUDENTS0=[
  {id:"s1",name:"Lucile",phone:"",email:"",address:"",pay:"Virement",notes:"",repertoire:""},
  {id:"s2",name:"Louis",phone:"",email:"",address:"",pay:"CESU+",notes:"",repertoire:""},
  {id:"s3",name:"Robinson & Telma",phone:"",email:"",address:"",pay:"Espèces",notes:"Duo cours partagés",repertoire:""},
  {id:"s4",name:"Riya",phone:"",email:"",address:"",pay:"CESU",notes:"Avancée, prépare concours",repertoire:""},
  {id:"s5",name:"Anne Marie",phone:"",email:"",address:"",pay:"Chèque",notes:"Adulte débutante",repertoire:""},
  {id:"s6",name:"David",phone:"",email:"",address:"",pay:"Virement",notes:"",repertoire:""},
];

const EVENTS0=[
  {id:1,title:"CDI Boutique Piano",date:"2026-03-01",time:"00:00",price:700,status:"Fait",paid:true,pm:"Virement",type:"work",origDate:null,sid:null},
  {id:2,title:"CDI Boutique Piano",date:"2026-04-01",time:"00:00",price:700,status:"Fait",paid:true,pm:"Virement",type:"work",origDate:null,sid:null},
  {id:3,title:"CDI Boutique Piano",date:"2026-05-01",time:"00:00",price:700,status:"Fait",paid:true,pm:"Virement",type:"work",origDate:null,sid:null},
  {id:4,title:"CDI Boutique Piano",date:"2026-06-01",time:"00:00",price:700,status:"Fait",paid:true,pm:"Virement",type:"work",origDate:null,sid:null},
  {id:5,title:"CDI Boutique Piano",date:"2026-07-01",time:"00:00",price:700,status:"Fait",paid:true,pm:"Virement",type:"work",origDate:null,sid:null},
  {id:6,title:"Conservatoire",date:"2026-03-02",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:7,title:"Robinson & Telma",date:"2026-03-03",time:"17:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:8,title:"Riya",date:"2026-03-03",time:"18:45",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:9,title:"Anne Marie",date:"2026-03-04",time:"15:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:10,title:"David",date:"2026-03-04",time:"17:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:11,title:"Travail",date:"2026-03-05",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:12,title:"Conservatoire",date:"2026-03-05",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:13,title:"Lucile",date:"2026-03-05",time:"18:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:14,title:"Conservatoire",date:"2026-03-05",time:"20:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:15,title:"Travail",date:"2026-03-06",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:16,title:"Louis",date:"2026-03-07",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:17,title:"Travail",date:"2026-03-07",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:18,title:"Prestation Ehpad",date:"2026-03-08",time:"14:00",price:0,status:"Prévu",paid:false,pm:null,type:"gig",origDate:null,sid:null},
  {id:19,title:"Conservatoire",date:"2026-03-09",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:20,title:"Robinson & Telma",date:"2026-03-10",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:21,title:"Riya",date:"2026-03-10",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:22,title:"Anne Marie",date:"2026-03-11",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:23,title:"David",date:"2026-03-11",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:24,title:"Travail",date:"2026-03-12",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:25,title:"Conservatoire",date:"2026-03-12",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:26,title:"Lucile",date:"2026-03-12",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:27,title:"Conservatoire",date:"2026-03-12",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:28,title:"Travail",date:"2026-03-13",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:29,title:"Louis",date:"2026-03-14",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:30,title:"Travail",date:"2026-03-14",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:31,title:"Conservatoire",date:"2026-03-16",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:32,title:"Robinson & Telma",date:"2026-03-17",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:33,title:"Riya",date:"2026-03-17",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:34,title:"Anne Marie",date:"2026-03-18",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:35,title:"David",date:"2026-03-18",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:36,title:"Travail",date:"2026-03-19",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:37,title:"Conservatoire",date:"2026-03-19",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:38,title:"Lucile",date:"2026-03-19",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:39,title:"Conservatoire",date:"2026-03-19",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:40,title:"Travail",date:"2026-03-20",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:41,title:"Louis",date:"2026-03-21",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:42,title:"Travail",date:"2026-03-21",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:43,title:"Conservatoire",date:"2026-03-23",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:44,title:"Robinson & Telma",date:"2026-03-24",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:45,title:"Riya",date:"2026-03-24",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:46,title:"Anne Marie",date:"2026-03-25",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:47,title:"David",date:"2026-03-25",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:48,title:"Travail",date:"2026-03-26",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:49,title:"Conservatoire",date:"2026-03-26",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:50,title:"Lucile",date:"2026-03-26",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:51,title:"Conservatoire",date:"2026-03-26",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:52,title:"Travail",date:"2026-03-27",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:53,title:"Louis",date:"2026-03-28",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:54,title:"Travail",date:"2026-03-28",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:55,title:"Conservatoire",date:"2026-03-30",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:56,title:"Robinson & Telma",date:"2026-03-31",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:57,title:"Riya",date:"2026-03-31",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:58,title:"Anne Marie",date:"2026-04-01",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:59,title:"David",date:"2026-04-01",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:60,title:"Travail",date:"2026-04-02",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:61,title:"Conservatoire",date:"2026-04-02",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:62,title:"Lucile",date:"2026-04-02",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:63,title:"Conservatoire",date:"2026-04-02",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:64,title:"Travail",date:"2026-04-03",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:65,title:"Louis",date:"2026-04-04",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:66,title:"Travail",date:"2026-04-04",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:67,title:"Prestation Ehpad",date:"2026-04-05",time:"14:00",price:0,status:"Prévu",paid:false,pm:null,type:"gig",origDate:null,sid:null},
  {id:68,title:"Conservatoire",date:"2026-04-06",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:69,title:"Robinson & Telma",date:"2026-04-07",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:70,title:"Riya",date:"2026-04-07",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:71,title:"Anne Marie",date:"2026-04-08",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:72,title:"David",date:"2026-04-08",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:73,title:"Travail",date:"2026-04-09",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:74,title:"Conservatoire",date:"2026-04-09",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:75,title:"Lucile",date:"2026-04-09",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:76,title:"Conservatoire",date:"2026-04-09",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:77,title:"Travail",date:"2026-04-10",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:78,title:"Louis",date:"2026-04-11",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:79,title:"Travail",date:"2026-04-11",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:80,title:"Conservatoire",date:"2026-04-13",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:81,title:"Robinson & Telma",date:"2026-04-14",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:82,title:"Riya",date:"2026-04-14",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:83,title:"Anne Marie",date:"2026-04-15",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:84,title:"David",date:"2026-04-15",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:85,title:"Travail",date:"2026-04-16",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:86,title:"Conservatoire",date:"2026-04-16",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:87,title:"Lucile",date:"2026-04-16",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:88,title:"Conservatoire",date:"2026-04-16",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:89,title:"Travail",date:"2026-04-17",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:90,title:"Louis",date:"2026-04-18",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:91,title:"Travail",date:"2026-04-18",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:92,title:"Conservatoire",date:"2026-04-20",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:93,title:"Robinson & Telma",date:"2026-04-21",time:"17:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:94,title:"Riya",date:"2026-04-21",time:"18:45",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:95,title:"Anne Marie",date:"2026-04-22",time:"15:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:96,title:"David",date:"2026-04-22",time:"17:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:97,title:"Travail",date:"2026-04-23",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:98,title:"Conservatoire",date:"2026-04-23",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:99,title:"Lucile",date:"2026-04-23",time:"18:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:100,title:"Conservatoire",date:"2026-04-23",time:"20:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:101,title:"Travail",date:"2026-04-24",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:102,title:"Louis",date:"2026-04-25",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:103,title:"Travail",date:"2026-04-25",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:104,title:"Conservatoire",date:"2026-04-27",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:105,title:"Robinson & Telma",date:"2026-04-28",time:"17:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:106,title:"Riya",date:"2026-04-28",time:"18:45",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:107,title:"Anne Marie",date:"2026-04-29",time:"15:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:108,title:"David",date:"2026-04-29",time:"17:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:109,title:"Travail",date:"2026-04-30",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:110,title:"Conservatoire",date:"2026-04-30",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:111,title:"Lucile",date:"2026-04-30",time:"18:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:112,title:"Conservatoire",date:"2026-04-30",time:"20:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:113,title:"Travail",date:"2026-05-01",time:"10:30",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:114,title:"Louis",date:"2026-05-02",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:115,title:"Travail",date:"2026-05-02",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:116,title:"Conservatoire",date:"2026-05-04",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:117,title:"Robinson & Telma",date:"2026-05-05",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:118,title:"Riya",date:"2026-05-05",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:119,title:"Anne Marie",date:"2026-05-06",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:120,title:"David",date:"2026-05-06",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:121,title:"Travail",date:"2026-05-07",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:122,title:"Conservatoire",date:"2026-05-07",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:123,title:"Lucile",date:"2026-05-07",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:124,title:"Conservatoire",date:"2026-05-07",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:125,title:"Travail",date:"2026-05-08",time:"10:30",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:126,title:"Louis",date:"2026-05-09",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:127,title:"Travail",date:"2026-05-09",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:128,title:"Prestation Ehpad",date:"2026-05-10",time:"14:00",price:0,status:"Prévu",paid:false,pm:null,type:"gig",origDate:null,sid:null},
  {id:129,title:"Conservatoire",date:"2026-05-11",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:130,title:"Robinson & Telma",date:"2026-05-12",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:131,title:"Riya",date:"2026-05-12",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:132,title:"Anne Marie",date:"2026-05-13",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:133,title:"David",date:"2026-05-13",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:134,title:"Travail",date:"2026-05-14",time:"10:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:135,title:"Conservatoire",date:"2026-05-14",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:136,title:"Lucile",date:"2026-05-14",time:"18:30",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:137,title:"Conservatoire",date:"2026-05-14",time:"20:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:138,title:"Travail",date:"2026-05-15",time:"10:30",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:139,title:"Louis",date:"2026-05-16",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:140,title:"Travail",date:"2026-05-16",time:"10:30",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:141,title:"Conservatoire",date:"2026-05-18",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:142,title:"Robinson & Telma",date:"2026-05-19",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:143,title:"Riya",date:"2026-05-19",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:144,title:"Anne Marie",date:"2026-05-20",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:145,title:"David",date:"2026-05-20",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:146,title:"Travail",date:"2026-05-21",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:147,title:"Conservatoire",date:"2026-05-21",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:148,title:"Lucile",date:"2026-05-21",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:149,title:"Conservatoire",date:"2026-05-21",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:150,title:"Travail",date:"2026-05-22",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:151,title:"Louis",date:"2026-05-23",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:152,title:"Travail",date:"2026-05-23",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:153,title:"Conservatoire",date:"2026-05-25",time:"15:00",price:0,status:"Vacances",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:154,title:"Robinson & Telma",date:"2026-05-26",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:155,title:"Riya",date:"2026-05-26",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:156,title:"Anne Marie",date:"2026-05-27",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:157,title:"David",date:"2026-05-27",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:158,title:"Travail",date:"2026-05-28",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:159,title:"Conservatoire",date:"2026-05-28",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:160,title:"Lucile",date:"2026-05-28",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:161,title:"Conservatoire",date:"2026-05-28",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:162,title:"Travail",date:"2026-05-29",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:163,title:"Louis",date:"2026-05-30",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:164,title:"Travail",date:"2026-05-30",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:165,title:"Conservatoire",date:"2026-06-01",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:166,title:"Robinson & Telma",date:"2026-06-02",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:167,title:"Riya",date:"2026-06-02",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:168,title:"Anne Marie",date:"2026-06-03",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:169,title:"David",date:"2026-06-03",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:170,title:"Travail",date:"2026-06-04",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:171,title:"Conservatoire",date:"2026-06-04",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:172,title:"Lucile",date:"2026-06-04",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:173,title:"Conservatoire",date:"2026-06-04",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:174,title:"Travail",date:"2026-06-05",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:175,title:"Louis",date:"2026-06-06",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:176,title:"Travail",date:"2026-06-06",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:177,title:"Prestation Ehpad",date:"2026-06-07",time:"14:00",price:0,status:"Prévu",paid:false,pm:null,type:"gig",origDate:null,sid:null},
  {id:178,title:"Conservatoire",date:"2026-06-08",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:179,title:"Robinson & Telma",date:"2026-06-09",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:180,title:"Riya",date:"2026-06-09",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:181,title:"Anne Marie",date:"2026-06-10",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:182,title:"David",date:"2026-06-10",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:183,title:"Travail",date:"2026-06-11",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:184,title:"Conservatoire",date:"2026-06-11",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:185,title:"Lucile",date:"2026-06-11",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:186,title:"Conservatoire",date:"2026-06-11",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:187,title:"Travail",date:"2026-06-12",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:188,title:"Louis",date:"2026-06-13",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:189,title:"Travail",date:"2026-06-13",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:190,title:"Conservatoire",date:"2026-06-15",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:191,title:"Robinson & Telma",date:"2026-06-16",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:192,title:"Riya",date:"2026-06-16",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:193,title:"Anne Marie",date:"2026-06-17",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:194,title:"David",date:"2026-06-17",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:195,title:"Travail",date:"2026-06-18",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:196,title:"Conservatoire",date:"2026-06-18",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:197,title:"Lucile",date:"2026-06-18",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:198,title:"Conservatoire",date:"2026-06-18",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:199,title:"Travail",date:"2026-06-19",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:200,title:"Louis",date:"2026-06-20",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:201,title:"Travail",date:"2026-06-20",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:202,title:"Conservatoire",date:"2026-06-22",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:203,title:"Robinson & Telma",date:"2026-06-23",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:204,title:"Riya",date:"2026-06-23",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:205,title:"Anne Marie",date:"2026-06-24",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:206,title:"David",date:"2026-06-24",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:207,title:"Travail",date:"2026-06-25",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:208,title:"Conservatoire",date:"2026-06-25",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:209,title:"Lucile",date:"2026-06-25",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:210,title:"Conservatoire",date:"2026-06-25",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:211,title:"Travail",date:"2026-06-26",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:212,title:"Louis",date:"2026-06-27",time:"09:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:213,title:"Travail",date:"2026-06-27",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:214,title:"Conservatoire",date:"2026-06-29",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:215,title:"Robinson & Telma",date:"2026-06-30",time:"17:00",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s3"},
  {id:216,title:"Riya",date:"2026-06-30",time:"18:45",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s4"},
  {id:217,title:"Anne Marie",date:"2026-07-01",time:"15:30",price:60,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s5"},
  {id:218,title:"David",date:"2026-07-01",time:"17:30",price:55,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s6"},
  {id:219,title:"Travail",date:"2026-07-02",time:"10:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:220,title:"Conservatoire",date:"2026-07-02",time:"15:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:221,title:"Lucile",date:"2026-07-02",time:"18:30",price:40,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:"s1"},
  {id:222,title:"Conservatoire",date:"2026-07-02",time:"20:00",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:223,title:"Travail",date:"2026-07-03",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null},
  {id:224,title:"Louis",date:"2026-07-04",time:"09:00",price:0,status:"Vacances",paid:false,pm:null,type:"lesson",origDate:null,sid:"s2"},
  {id:225,title:"Travail",date:"2026-07-04",time:"10:30",price:0,status:"Prévu",paid:false,pm:null,type:"personal",origDate:null,sid:null}
];

const RECS0=[
  {id:"rlu",title:"Lucile",time:"18:30",price:40,type:"lesson",dow:4,active:true,sid:"s1"},
  {id:"rl",title:"Louis",time:"09:00",price:50,type:"lesson",dow:6,active:true,sid:"s2"},
  {id:"rrt",title:"Robinson & Telma",time:"17:00",price:40,type:"lesson",dow:2,active:true,sid:"s3"},
  {id:"rri",title:"Riya",time:"18:45",price:50,type:"lesson",dow:2,active:true,sid:"s4"},
  {id:"ram",title:"Anne Marie",time:"15:30",price:60,type:"lesson",dow:3,active:true,sid:"s5"},
  {id:"rdv",title:"David",time:"17:30",price:55,type:"lesson",dow:3,active:true,sid:"s6"},
];

function genRec(recs,month,year,existing){
  const out=[],dim=new Date(year,month,0).getDate();
  recs.filter(r=>r.active).forEach(r=>{
    for(let d=1;d<=dim;d++){
      const dt=new Date(year,month-1,d);
      if(dt.getDay()===r.dow){
        const ds=year+"-"+String(month).padStart(2,"0")+"-"+String(d).padStart(2,"0");
        if(!existing.some(e=>e.date===ds&&e.title===r.title))
          out.push({id:_ID++,title:r.title,date:ds,time:r.time,price:r.price,status:"Prévu",paid:false,pm:null,type:r.type,origDate:null,sid:r.sid||null});
      }
    }
  });
  return out;
}

const THEMES={
  // 🌙 Original — légèrement plus opaque/saturé
  dark:{name:"🌙 Sombre",bg:"#070712",phone:"#0c0c1a",card:"rgba(255,255,255,0.07)",border:"rgba(255,255,255,0.12)",text:"#f2f2fc",muted:"rgba(242,242,252,0.55)",faint:"rgba(242,242,252,0.28)",gold:"#fbbf24",gold2:"#f59e0b",accent:"#6366f1",accent2:"#8b5cf6",green:"#34d399",red:"#f87171",blue:"#60a5fa",nav:"rgba(7,7,18,0.98)",navB:"rgba(255,255,255,0.1)",inp:"rgba(255,255,255,0.09)",inpB:"rgba(255,255,255,0.15)",sheet:"#111128",outer:"radial-gradient(ellipse at 30% -20%,rgba(99,102,241,0.28) 0%,transparent 55%),radial-gradient(ellipse at 70% 110%,rgba(251,191,36,0.12) 0%,transparent 55%),#070712",sec:"rgba(242,242,252,0.38)",prog:"rgba(255,255,255,0.1)",calE:"rgba(0,0,0,0.25)",calB:"rgba(255,255,255,0.06)",calH:"rgba(255,255,255,0.04)",hi:"rgba(99,102,241,0.2)"},

  // ☀️ Ciel du jour — nuages, soleil, bleu ciel vif
  sky:{name:"☀️ Ciel",bg:"#e0f2fe",phone:"#f0faff",card:"rgba(255,255,255,0.88)",border:"rgba(56,189,248,0.45)",text:"#0c3247",muted:"rgba(12,50,71,0.6)",faint:"rgba(12,50,71,0.32)",gold:"#f59e0b",gold2:"#d97706",accent:"#0284c7",accent2:"#0ea5e9",green:"#059669",red:"#dc2626",blue:"#0284c7",nav:"rgba(224,242,254,0.97)",navB:"rgba(56,189,248,0.35)",inp:"rgba(255,255,255,0.9)",inpB:"rgba(56,189,248,0.5)",sheet:"#f0f9ff",outer:"radial-gradient(ellipse at 20% -30%,rgba(253,224,71,0.55) 0%,transparent 40%),radial-gradient(ellipse at 80% -10%,rgba(56,189,248,0.35) 0%,transparent 45%),linear-gradient(180deg,#bae6fd 0%,#e0f2fe 50%,#f0f9ff 100%)",sec:"rgba(12,50,71,0.45)",prog:"rgba(56,189,248,0.25)",calE:"rgba(219,234,254,0.5)",calB:"rgba(56,189,248,0.18)",calH:"rgba(240,249,255,0.8)",hi:"rgba(2,132,199,0.12)",clouds:true},

  // 🌸 Rose sakura
  rose:{name:"🌸 Sakura",bg:"#fce7f3",phone:"#fff0f8",card:"rgba(255,255,255,0.88)",border:"rgba(249,168,212,0.55)",text:"#831843",muted:"rgba(131,24,67,0.58)",faint:"rgba(131,24,67,0.32)",gold:"#db2777",gold2:"#be185d",accent:"#ec4899",accent2:"#f43f5e",green:"#059669",red:"#dc2626",blue:"#7c3aed",nav:"rgba(255,240,248,0.97)",navB:"rgba(249,168,212,0.45)",inp:"rgba(255,255,255,0.85)",inpB:"rgba(249,168,212,0.65)",sheet:"#fff0f8",outer:"radial-gradient(ellipse at 30% -10%,rgba(236,72,153,0.22) 0%,transparent 55%),radial-gradient(ellipse at 80% 110%,rgba(244,63,94,0.15) 0%,transparent 55%),#fce7f3",sec:"rgba(131,24,67,0.45)",prog:"rgba(249,168,212,0.35)",calE:"rgba(252,231,243,0.6)",calB:"rgba(249,168,212,0.28)",calH:"rgba(255,240,248,0.85)",hi:"rgba(236,72,153,0.14)"},

  // ⚡ Cyberpunk — néon violet/cyan sur noir profond
  cyber:{name:"⚡ Cyber",bg:"#060612",phone:"#0a0a1e",card:"rgba(139,92,246,0.08)",border:"rgba(139,92,246,0.2)",text:"#f0e6ff",muted:"rgba(240,230,255,0.5)",faint:"rgba(240,230,255,0.25)",gold:"#f0abfc",gold2:"#e879f9",accent:"#a855f7",accent2:"#06b6d4",green:"#22d3ee",red:"#f43f5e",blue:"#06b6d4",nav:"rgba(6,6,18,0.98)",navB:"rgba(139,92,246,0.18)",inp:"rgba(139,92,246,0.07)",inpB:"rgba(139,92,246,0.25)",sheet:"#0d0d22",outer:"radial-gradient(ellipse at 20% 0%,rgba(168,85,247,0.3) 0%,transparent 50%),radial-gradient(ellipse at 80% 100%,rgba(6,182,212,0.25) 0%,transparent 50%),#060612",sec:"rgba(240,230,255,0.38)",prog:"rgba(139,92,246,0.15)",calE:"rgba(0,0,0,0.3)",calB:"rgba(139,92,246,0.1)",calH:"rgba(139,92,246,0.06)",hi:"rgba(168,85,247,0.18)"},

  // 🌿 Forêt profonde
  forest:{name:"🌿 Forêt",bg:"#052e16",phone:"#041f0f",card:"rgba(255,255,255,0.06)",border:"rgba(74,222,128,0.18)",text:"#dcfce7",muted:"rgba(220,252,231,0.55)",faint:"rgba(220,252,231,0.28)",gold:"#fbbf24",gold2:"#f59e0b",accent:"#4ade80",accent2:"#22c55e",green:"#86efac",red:"#fca5a5",blue:"#93c5fd",nav:"rgba(5,46,22,0.98)",navB:"rgba(74,222,128,0.14)",inp:"rgba(255,255,255,0.06)",inpB:"rgba(74,222,128,0.22)",sheet:"#071c0e",outer:"radial-gradient(ellipse at 30% -20%,rgba(74,222,128,0.18) 0%,transparent 55%),radial-gradient(ellipse at 80% 110%,rgba(34,197,94,0.12) 0%,transparent 55%),#052e16",sec:"rgba(220,252,231,0.4)",prog:"rgba(74,222,128,0.12)",calE:"rgba(0,0,0,0.25)",calB:"rgba(74,222,128,0.1)",calH:"rgba(74,222,128,0.06)",hi:"rgba(74,222,128,0.14)"},

  // ✨ Or & Noir
  gold:{name:"✨ Or & Noir",bg:"#0a0800",phone:"#130f00",card:"rgba(251,191,36,0.06)",border:"rgba(251,191,36,0.15)",text:"#fef3c7",muted:"rgba(254,243,199,0.55)",faint:"rgba(254,243,199,0.28)",gold:"#fbbf24",gold2:"#f59e0b",accent:"#fbbf24",accent2:"#f59e0b",green:"#34d399",red:"#f87171",blue:"#60a5fa",nav:"rgba(10,8,0,0.98)",navB:"rgba(251,191,36,0.12)",inp:"rgba(251,191,36,0.06)",inpB:"rgba(251,191,36,0.18)",sheet:"#100d00",outer:"radial-gradient(ellipse at 30% -20%,rgba(251,191,36,0.18) 0%,transparent 55%),radial-gradient(ellipse at 80% 110%,rgba(245,158,11,0.1) 0%,transparent 55%),#0a0800",sec:"rgba(254,243,199,0.4)",prog:"rgba(251,191,36,0.1)",calE:"rgba(0,0,0,0.35)",calB:"rgba(251,191,36,0.08)",calH:"rgba(251,191,36,0.05)",hi:"rgba(251,191,36,0.12)"},
};
let T=THEMES.dark;
let G="linear-gradient(135deg,#6366f1,#8b5cf6)";
const GG="linear-gradient(135deg,#fbbf24,#f59e0b)";

// Fonction mail sécurisée — <a> invisible pour ne pas recharger la page sur Samsung
function safeMail(subject,body,toastCb){
  const openMailto=(subj,bd)=>{
    const a=document.createElement("a");
    a.href="mailto:?subject="+encodeURIComponent(subj)+(bd?"&body="+encodeURIComponent(bd):"");
    a.style.cssText="position:fixed;top:-100px;left:-100px;opacity:0;";
    document.body.appendChild(a);
    a.click();
    setTimeout(()=>{ try{document.body.removeChild(a);}catch(e){} },1000);
  };
  if(encodeURIComponent(body).length<=600){
    openMailto(subject,body);
  } else {
    const tryClip=navigator.clipboard&&navigator.clipboard.writeText;
    if(tryClip){
      navigator.clipboard.writeText(body).then(()=>{
        if(toastCb) toastCb("📋 Texte copié ! Colle-le dans ton mail.");
        setTimeout(()=>openMailto(subject,null),350);
      }).catch(()=>openMailto(subject,body.slice(0,500)));
    } else {
      openMailto(subject,body.slice(0,500));
    }
  }
}

const MailToast=({msg})=>msg?(<div style={{position:"fixed",bottom:90,left:"50%",transform:"translateX(-50%)",background:"rgba(30,30,50,0.97)",border:"1px solid rgba(99,102,241,0.4)",borderRadius:14,padding:"11px 18px",fontSize:12,color:"#f0f0f8",zIndex:200,whiteSpace:"nowrap",boxShadow:"0 8px 24px rgba(0,0,0,0.4)"}}>{msg}</div>):null;

const Overlay=({onClose,children,z=50})=>(
  <div onClick={onClose} style={{position:"absolute",inset:0,background:"rgba(0,0,0,0.7)",backdropFilter:"blur(14px)",zIndex:z,display:"flex",flexDirection:"column",justifyContent:"flex-end"}}>
    {children}
  </div>
);
const Sheet=({children,full})=>(
  <div onClick={e=>e.stopPropagation()} style={{background:T.sheet,borderRadius:"28px 28px 0 0",maxHeight:full?"93%":"88%",height:full?"93%":undefined,display:"flex",flexDirection:"column",animation:"slideUp .28s cubic-bezier(.4,0,.2,1)",overflow:"hidden",border:"1px solid rgba(255,255,255,0.06)",borderBottom:"none"}}>
    <div style={{width:36,height:4,background:"rgba(255,255,255,0.12)",borderRadius:2,margin:"12px auto 0",flexShrink:0}}/>
    {children}
  </div>
);
const SHdr=({children})=><div style={{padding:"12px 20px 8px",flexShrink:0}}>{children}</div>;
const SBody=({children})=><div style={{flex:1,overflowY:"auto",padding:"4px 20px 32px"}}>{children}</div>;
const SLbl=({text,mt=0,mb=8,color})=><div style={{fontSize:10,fontWeight:600,letterSpacing:"0.7px",textTransform:"uppercase",color:color||T.sec,margin:mt+"px 0 "+mb+"px"}}>{text}</div>;
const FLbl=({text})=><div style={{fontSize:10,fontWeight:600,letterSpacing:"0.6px",textTransform:"uppercase",color:T.sec,marginBottom:6}}>{text}</div>;
const CBtn=({onClose})=><button onClick={onClose} style={{background:T.card,border:"1px solid "+T.border,borderRadius:9,width:30,height:30,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:T.muted,fontSize:18,flexShrink:0,lineHeight:1,fontFamily:"DM Sans,sans-serif"}}>×</button>;
const Inp=({...p})=><input {...p} style={{width:"100%",background:T.inp,border:"1px solid "+T.inpB,borderRadius:12,padding:"11px 13px",color:T.text,fontSize:13,outline:"none",marginBottom:10,fontFamily:"DM Sans,sans-serif",...(p.style||{})}}/>;
const Sel=({children,...p})=><select {...p} style={{width:"100%",background:T.sheet,border:"1px solid "+T.inpB,borderRadius:12,padding:"11px 13px",color:T.text,fontSize:13,outline:"none",marginBottom:10,fontFamily:"DM Sans,sans-serif",...(p.style||{})}}>{children}</select>;
const Btn=({children,onClick,style={}})=><button onClick={onClick} style={{background:G,border:"none",borderRadius:12,padding:"11px 18px",color:"white",fontWeight:600,fontSize:13,cursor:"pointer",fontFamily:"DM Sans,sans-serif",...style}}>{children}</button>;

function XPBar({enc,paid,target,pct}){
  const isMax=pct>=100;
  return(
    <div style={{background:"rgba(251,191,36,0.06)",border:"1px solid rgba(251,191,36,0.2)",borderRadius:16,padding:"12px 14px",marginBottom:10,animation:isMax?"goldGlow 2s infinite":undefined}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:8}}>
        <div>
          <div style={{display:"flex",alignItems:"center",gap:6,marginBottom:4}}>
            <span style={{fontSize:13}}>⚔️</span>
            <span style={{fontSize:10,fontWeight:700,color:T.gold,letterSpacing:"0.5px"}}>QUÊTE DU MOIS</span>
          </div>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:32,color:isMax?T.gold:T.text,lineHeight:1}}>{enc}<span style={{fontSize:14,color:T.muted}}>€</span></div>
          <div style={{fontSize:9,color:T.faint,marginTop:2}}>total activités du mois</div>
        </div>
        <div style={{textAlign:"right"}}>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.muted,lineHeight:1}}>{target}<span style={{fontSize:11}}>€</span></div>
          <div style={{fontSize:9,color:T.faint,marginTop:2}}>objectif récurrences</div>
          {paid>0&&<div style={{fontSize:9,color:T.green,marginTop:3,fontWeight:700}}>✓ {paid}€ encaissés</div>}
        </div>
      </div>
      <div style={{height:7,background:"rgba(251,191,36,0.1)",borderRadius:4,overflow:"hidden",marginBottom:5}}>
        <div style={{height:"100%",borderRadius:4,background:isMax?"linear-gradient(90deg,#fbbf24,#f59e0b)":G,width:Math.min(pct,100)+"%",transition:"width .6s ease"}}/>
      </div>
      <div style={{display:"flex",justifyContent:"space-between",fontSize:9,color:T.faint}}>
        <span>{pct}%{pct>100?" 🔥":""}</span>
        <span style={{color:isMax?T.gold:T.faint}}>{isMax?"🏆 Objectif dépassé !":target>0&&enc<target?(target-enc)+"€ restants":""}</span>
      </div>
    </div>
  );
}

function QCard({ev,onEdit,onRelance}){
  const qs=questStatus(ev);
  const tc=TC[ev.type]||TC.lesson;
  const isAlert=qs.key==="Alerte";
  const isOk=qs.key==="Accomplie";
  const isTask=ev.type==="task";
  if(isTask) return(
    <div onClick={()=>onEdit({...ev})} style={{background:tc.color+"12",border:"1px solid "+tc.color+"40",borderRadius:12,padding:"9px 13px",cursor:"pointer",display:"flex",alignItems:"center",gap:10,marginBottom:6}}>
      <div style={{width:30,height:30,borderRadius:9,background:tc.color+"25",border:"1px solid "+tc.color+"50",display:"flex",alignItems:"center",justifyContent:"center",fontSize:14,flexShrink:0}}>{tc.icon}</div>
      <div style={{flex:1,minWidth:0}}>
        <span style={{fontSize:12,fontWeight:600,color:T.text,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap",display:"block"}}>{ev.title}</span>
        <span style={{fontSize:9,color:T.faint}}>⏱ {fd(ev.date)} · {ev.time}</span>
      </div>
    </div>
  );
  return(
    <div onClick={()=>onEdit({...ev})} style={{background:T.card,border:"1px solid "+(isAlert?"rgba(248,113,113,0.3)":isOk?"rgba(251,191,36,0.2)":tc.color+"30"),borderRadius:16,padding:"11px 13px",cursor:"pointer",display:"flex",alignItems:"center",gap:10,marginBottom:8,position:"relative",overflow:"hidden"}}>
      {/* Barre couleur type à gauche */}
      <div style={{position:"absolute",left:0,top:0,bottom:0,width:3,background:tc.color,borderRadius:"3px 0 0 3px"}}/>
      {isOk&&<div style={{position:"absolute",top:0,left:0,right:0,height:2,background:GG}}/>}
      <div style={{width:36,height:36,borderRadius:11,background:tc.color+"20",border:"1px solid "+tc.color+"45",display:"flex",alignItems:"center",justifyContent:"center",fontSize:18,flexShrink:0,marginLeft:4}}>{tc.icon}</div>
      <div style={{flex:1,minWidth:0}}>
        <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:3}}>
          <span style={{fontSize:13,fontWeight:600,color:T.text,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap",maxWidth:155}}>{ev.title}</span>
          <span style={{fontFamily:"DM Serif Display,serif",fontSize:15,color:isOk?T.gold:ev.price>0?T.text:T.muted,flexShrink:0,marginLeft:6}}>{ev.price>0?ev.price+"€":"—"}</span>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:6,flexWrap:"wrap"}}>
          <span style={{fontSize:9,padding:"1px 6px",borderRadius:20,background:tc.color+"18",color:tc.color,border:"1px solid "+tc.color+"40",fontWeight:600}}>{tc.label}</span>
          <span style={{fontSize:9,color:T.muted}}>⏱ {fd(ev.date)} · {ev.time}</span>
          <span style={{display:"inline-flex",alignItems:"center",gap:3,padding:"2px 7px",borderRadius:20,fontSize:9,fontWeight:600,border:"1px solid",background:qs.bg,color:qs.col,borderColor:qs.b}}>{qs.ico} {qs.label}</span>
          {ev.origDate&&<span style={{fontSize:9,color:T.gold}}>📅 était le {fd(ev.origDate)}</span>}
        </div>
      </div>
      {isAlert&&(
        <div onClick={e=>{e.stopPropagation();onRelance(ev);}} style={{flexShrink:0,background:"rgba(248,113,113,0.15)",border:"1px solid rgba(248,113,113,0.3)",borderRadius:9,padding:"5px 9px",fontSize:10,fontWeight:600,color:T.red,cursor:"pointer",whiteSpace:"nowrap"}}>📱 Relance</div>
      )}
    </div>
  );
}

// ══ PLANNING TAB ══
function PlanningTab({monthEvs,sorted,stats,pct,setPanel,alertCount,setAlerts,setRecPanel,recs,setEditing,onRelance}){
  // Filtrer : seuls les événements avec price > 0 dans les quêtes
  const money=sorted.filter(e=>e.price>0);
  const groups={
    alert:money.filter(e=>questStatus(e).key==="Alerte"),
    route:money.filter(e=>questStatus(e).key==="EnRoute"),
    plan:money.filter(e=>questStatus(e).key==="Aventure"),
    done:money.filter(e=>questStatus(e).key==="Accomplie"),
    // Pertes & Rattrapages = annulé avec prix + à rattraper avec prix
    pertes:money.filter(e=>questStatus(e).key==="Rattrapage"||questStatus(e).key==="Annulé"),
  };
  // XPBar : objectif = CA prévu (stats.prev), encaissé = stats.enc
  const xpPct=stats.prev>0?Math.min(Math.round(stats.enc/stats.prev*100),999):stats.enc>0?100:0;
  // Butin en attente = Fait + non payé + prix > 0
  const butinDue=money.filter(e=>e.status==="Fait"&&!e.paid);
  const butinTotal=butinDue.reduce((a,e)=>a+e.price,0);
  return(<>
    <XPBar enc={stats.enc} paid={stats.enc} target={stats.prev} pct={xpPct}/>
    <div style={{display:"flex",gap:8,marginBottom:14,overflowX:"auto",paddingBottom:2}}>
      {[
        {l:"🚨 Alertes"+(alertCount>0?" ("+alertCount+")":""),a:()=>setAlerts(true),c:alertCount>0?T.red:T.muted,bc:alertCount>0?"rgba(248,113,113,0.3)":T.border},
        {l:"🔁 Récurrences ("+recs.filter(r=>r.active).length+")",a:()=>setRecPanel(true),c:T.green,bc:"rgba(52,211,153,0.3)"},
      ].map((b,i)=>(
        <div key={i} onClick={b.a} style={{display:"flex",alignItems:"center",gap:6,padding:"8px 13px",background:T.card,border:"1px solid "+b.bc,borderRadius:20,fontSize:11,fontWeight:500,color:b.c,cursor:"pointer",whiteSpace:"nowrap",flexShrink:0}}>{b.l}</div>
      ))}
    </div>
    <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:10,marginBottom:14}}>
      {[
        {id:"ca",lbl:"CA Prévu",val:stats.prev+"€",sub:stats.total+" cours",bar:G},
        {id:"enc",lbl:"Encaissé",val:stats.enc+"€",sub:xpPct+"% du prévu",bar:"linear-gradient(90deg,"+T.green+",transparent)",vc:T.green},
        {id:"due",lbl:"Butin en attente",val:butinTotal+"€",sub:butinDue.length+" cours faits",bar:butinTotal>0?"linear-gradient(90deg,"+T.gold+",transparent)":undefined,vc:butinTotal>0?T.gold:undefined},
        {id:"rat",lbl:"Pertes & Rattrapages",val:String(groups.pertes.length),sub:"annulés / décalés",bar:groups.pertes.length>0?"linear-gradient(90deg,"+T.red+",transparent)":undefined,vc:groups.pertes.length>0?T.red:undefined},
      ].map(t=>(
        <div key={t.id} onClick={()=>setPanel(t.id)} style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 15px",cursor:"pointer",position:"relative",overflow:"hidden"}}>
          <div style={{position:"absolute",top:8,right:10,fontSize:8,color:T.faint}}>›</div>
          <SLbl text={t.lbl} mb={6}/>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:26,lineHeight:1,color:t.vc||T.text}}>{t.val}</div>
          <div style={{fontSize:10,color:T.faint,marginTop:4}}>{t.sub}</div>
          <div style={{position:"absolute",bottom:0,left:0,right:0,height:2.5,background:t.bar||"transparent"}}/>
        </div>
      ))}
    </div>
    {groups.alert.length>0&&<><SLbl text="🚨 Alerte Butin — Relance urgente" color={T.red} mb={9}/>{groups.alert.map(e=><QCard key={e.id} ev={e} onEdit={setEditing} onRelance={onRelance}/>)}</>}
    {groups.route.length>0&&<><SLbl text="📦 Butin en Route" color={T.blue} mt={groups.alert.length>0?10:0} mb={9}/>{groups.route.map(e=><QCard key={e.id} ev={e} onEdit={setEditing} onRelance={onRelance}/>)}</>}
    {groups.plan.length>0&&<><SLbl text="🗺️ Prochaines Aventures" mt={10} mb={9}/>{groups.plan.map(e=><QCard key={e.id} ev={e} onEdit={setEditing} onRelance={onRelance}/>)}</>}
    {groups.done.length>0&&<>
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",margin:"10px 0 9px"}}>
        <SLbl text={"⚔️ Quêtes Accomplies ("+groups.done.length+")"} color={T.gold} mb={0}/>
        <span style={{fontSize:9,color:T.faint}}>appuie pour modifier</span>
      </div>
      {groups.done.map(e=><QCard key={e.id} ev={e} onEdit={setEditing} onRelance={onRelance}/>)}
    </>}
    {groups.pertes.length>0&&<>
      <div style={{height:1,background:T.border,margin:"12px 0 10px"}}/>
      <SLbl text="⚠️ Pertes & Rattrapages" color={T.red} mb={9}/>
      {groups.pertes.map(e=><QCard key={e.id} ev={e} onEdit={setEditing} onRelance={onRelance}/>)}
    </>}
    {money.length===0&&<div style={{textAlign:"center",padding:"32px 0",color:T.faint,fontSize:12}}>Aucune activité rémunérée ce mois</div>}
  </>);
}

function CalendarTab({events,setEvents,month,year,onEdit}){
  const [view,setView]=useState("month");
  const [dragId,setDragId]=useState(null);
  const [dragOver,setDragOver]=useState(null);
  const [flash,setFlash]=useState(null);
  const [wkStart,setWkStart]=useState(()=>{const d=new Date(year,month-1,1);d.setDate(d.getDate()-d.getDay());return d;});
  const today=new Date();
  const todayStr=today.getFullYear()+"-"+String(today.getMonth()+1).padStart(2,"0")+"-"+String(today.getDate()).padStart(2,"0");
  const cells=useMemo(()=>{const fd=new Date(year,month-1,1).getDay(),dim=new Date(year,month,0).getDate();const c=Array(fd).fill(null);for(let d=1;d<=dim;d++){const ds=year+"-"+String(month).padStart(2,"0")+"-"+String(d).padStart(2,"0");c.push({day:d,ds,evs:events.filter(e=>e.date===ds).sort((a,b)=>a.time.localeCompare(b.time))});}return c;},[events,month,year]);
  const wkCells=useMemo(()=>{const c=[];for(let i=0;i<7;i++){const d=new Date(wkStart);d.setDate(wkStart.getDate()+i);const ds=d.getFullYear()+"-"+String(d.getMonth()+1).padStart(2,"0")+"-"+String(d.getDate()).padStart(2,"0");c.push({day:d.getDate(),ds,dn:DFR[d.getDay()],evs:events.filter(e=>e.date===ds).sort((a,b)=>a.time.localeCompare(b.time))});}return c;},[wkStart,events]);
  const pW=()=>{const d=new Date(wkStart);d.setDate(d.getDate()-7);setWkStart(d);};
  const nW=()=>{const d=new Date(wkStart);d.setDate(d.getDate()+7);setWkStart(d);};
  const onDS=(e,ev)=>{setDragId(ev.id);e.dataTransfer.effectAllowed="move";};
  const onDE=()=>{setDragId(null);setDragOver(null);};
  const onDO=(e,ds)=>{e.preventDefault();setDragOver(ds);};
  const onDrop=(e,ds)=>{e.preventDefault();if(dragId){setEvents(p=>p.map(ev=>ev.id===dragId?{...ev,date:ds}:ev));setFlash(ds);setTimeout(()=>setFlash(null),700);}setDragId(null);setDragOver(null);};
  const CEv=({ev})=>{const tc=TC[ev.type]||TC.lesson;const isNow=ev.date===todayStr;return(<div draggable onDragStart={e=>onDS(e,ev)} onDragEnd={onDE} onClick={e=>{e.stopPropagation();onEdit({...ev});}} style={{background:isNow?tc.color+"28":tc.color+"15",borderLeft:"2px solid "+tc.color,borderRadius:"0 4px 4px 0",padding:"2px 4px",marginBottom:2,cursor:"grab",opacity:dragId===ev.id?0.3:1,boxShadow:isNow?"0 0 8px "+tc.color+"55":"none",overflow:"hidden"}}><div style={{fontSize:8,fontWeight:600,color:isNow?tc.color:tc.color+"bb",lineHeight:1.3,whiteSpace:"nowrap",overflow:"hidden",textOverflow:"ellipsis"}}>{ev.time.slice(0,5)} {ev.title.split(" ")[0]}</div></div>);};
  return(<>
    <div style={{display:"flex",marginBottom:14,background:T.calE,border:"1px solid "+T.border,borderRadius:14,padding:4}}>
      {[["month","📅 Mois"],["week","📆 Semaine"]].map(([v,l])=>(
        <div key={v} onClick={()=>setView(v)} style={{flex:1,padding:"8px",borderRadius:10,textAlign:"center",fontSize:12,fontWeight:600,cursor:"pointer",background:view===v?G:"transparent",color:view===v?"white":T.muted,transition:"all .2s"}}>{l}</div>
      ))}
    </div>
    {view==="month"&&<>
      <div style={{fontSize:10,color:T.faint,textAlign:"center",marginBottom:8}}>✦ Glisse un cours pour changer de date</div>
      <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:20,overflow:"hidden"}}>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)",background:T.calH,borderBottom:"1px solid "+T.calB}}>
          {DFR.map(d=><div key={d} style={{padding:"9px 0",textAlign:"center",fontSize:9,fontWeight:700,letterSpacing:"0.7px",textTransform:"uppercase",color:T.muted}}>{d}</div>)}
        </div>
        <div style={{display:"grid",gridTemplateColumns:"repeat(7,1fr)"}}>
          {cells.map((cell,i)=>{
            if(!cell) return <div key={"e"+i} style={{height:70,background:T.calE,borderRight:"1px solid "+T.calB,borderBottom:"1px solid "+T.calB}}/>;
            const isTd=cell.ds===todayStr,isDO=dragOver===cell.ds,fl=flash===cell.ds;
            const hasAlert=cell.evs.some(e=>questStatus(e).key==="Alerte");
            return(<div key={cell.ds} onDragOver={e=>onDO(e,cell.ds)} onDrop={e=>onDrop(e,cell.ds)} style={{height:70,padding:"3px",borderRight:"1px solid "+T.calB,borderBottom:"1px solid "+T.calB,background:fl?"rgba(99,102,241,0.2)":isDO?"rgba(99,102,241,0.12)":isTd?T.hi:"transparent",outline:isDO?"2px dashed "+T.accent:fl?"2px solid "+T.accent+"55":"none",outlineOffset:"-2px",position:"relative",overflow:"hidden",transition:"background .15s"}}>
              <div style={{display:"flex",alignItems:"center",justifyContent:"center",width:18,height:18,borderRadius:"50%",background:isTd?G:"transparent",marginBottom:2,boxShadow:isTd?"0 0 8px "+T.accent+"66":"none"}}>
                <span style={{fontSize:9,color:isTd?"white":T.muted,fontWeight:isTd?700:400,lineHeight:1}}>{cell.day}</span>
              </div>
              {hasAlert&&<div style={{position:"absolute",top:3,right:3,width:5,height:5,borderRadius:"50%",background:T.red,animation:"pulseR 1.5s infinite"}}/>}
              {cell.evs.slice(0,2).map(ev=><CEv key={ev.id} ev={ev}/>)}
              {cell.evs.length>2&&<div style={{fontSize:7,color:T.accent,paddingLeft:3,fontWeight:600}}>+{cell.evs.length-2}</div>}
            </div>);
          })}
        </div>
      </div>
      <div style={{display:"flex",gap:8,flexWrap:"wrap",marginTop:10,padding:"9px 12px",background:T.card,border:"1px solid "+T.border,borderRadius:12}}>
        {Object.entries(TC).map(([k,v])=><div key={k} style={{display:"flex",alignItems:"center",gap:5,fontSize:9,color:T.muted}}><span style={{width:7,height:7,borderRadius:2,background:v.color,display:"inline-block"}}/>{v.label}</div>)}
        <div style={{marginLeft:"auto",fontSize:9,color:T.faint}}>🔴 = alerte paiement</div>
      </div>
    </>}
    {view==="week"&&<>
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:12,background:T.card,border:"1px solid "+T.border,borderRadius:13,padding:"10px 16px"}}>
        <button onClick={pW} style={{background:"none",border:"none",color:T.muted,cursor:"pointer",fontSize:20,lineHeight:1}}>‹</button>
        <span style={{fontSize:12,fontWeight:600,color:T.text}}>{wkCells[0]&&fd(wkCells[0].ds)} – {wkCells[6]&&fd(wkCells[6].ds)}</span>
        <button onClick={nW} style={{background:"none",border:"none",color:T.muted,cursor:"pointer",fontSize:20,lineHeight:1}}>›</button>
      </div>
      {wkCells.map(cell=>{const isTd=cell.ds===todayStr;return(
        <div key={cell.ds} style={{background:isTd?T.hi:T.card,border:"1px solid "+(isTd?T.accent+"55":T.border),borderRadius:16,overflow:"hidden",marginBottom:8,boxShadow:isTd?"0 0 16px "+T.accent+"22":"none"}}>
          <div style={{display:"flex",alignItems:"center",gap:10,padding:"10px 14px",borderBottom:cell.evs.length>0?"1px solid "+T.calB:"none"}}>
            <div style={{width:36,height:36,borderRadius:11,background:isTd?G:T.calH,display:"flex",flexDirection:"column",alignItems:"center",justifyContent:"center",flexShrink:0}}>
              <span style={{fontSize:8,color:isTd?"rgba(255,255,255,0.7)":T.faint,lineHeight:1}}>{cell.dn}</span>
              <span style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:isTd?"white":T.text,lineHeight:1.1}}>{cell.day}</span>
            </div>
            <span style={{fontSize:11,color:isTd?T.accent:T.muted,fontWeight:isTd?600:400}}>{cell.evs.length===0?"Rien de prévu":cell.evs.length+" événement"+(cell.evs.length>1?"s":"")}</span>
            {isTd&&<span style={{marginLeft:"auto",fontSize:9,padding:"2px 8px",borderRadius:20,background:T.accent+"22",color:T.accent,border:"1px solid "+T.accent+"44",fontWeight:600}}>Aujourd'hui</span>}
          </div>
          {cell.evs.map(ev=>{const tc=TC[ev.type]||TC.lesson;const qs=questStatus(ev);return(
            <div key={ev.id} draggable onDragStart={e=>onDS(e,ev)} onDragEnd={onDE} onClick={()=>onEdit({...ev})} style={{display:"flex",alignItems:"center",gap:11,padding:"10px 14px",borderBottom:"1px solid "+T.calB,cursor:"grab",background:tc.color+"08"}}>
              <div style={{width:3,alignSelf:"stretch",background:tc.color,borderRadius:3,flexShrink:0}}/>
              <div style={{width:24,height:24,borderRadius:7,background:tc.color+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:13,flexShrink:0}}>{tc.icon}</div>
              <div style={{width:38,flexShrink:0}}><div style={{fontSize:12,fontWeight:600,color:T.text}}>{ev.time}</div></div>
              <div style={{flex:1,minWidth:0}}>
                <div style={{fontSize:13,fontWeight:500,color:T.text,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{ev.title}</div>
                {ev.origDate&&<div style={{fontSize:9,color:T.gold,marginTop:2}}>🔄 était le {fd(ev.origDate)}</div>}
              </div>
              <div style={{flexShrink:0,textAlign:"right"}}>
                <span style={{fontSize:9,padding:"2px 7px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b}}>{qs.ico} {qs.label}</span>
                {ev.price>0&&<div style={{fontFamily:"DM Serif Display,serif",fontSize:13,color:ev.paid?T.green:T.text,marginTop:3}}>{ev.price}€</div>}
              </div>
            </div>);})}
        </div>);})}
    </>}
  </>);
}

function StudentsTab({students,events,onFiche,month,year}){
  const [q,setQ]=useState("");
  const list=students.filter(s=>s.name.toLowerCase().includes(q.toLowerCase()));
  const avC=["#818cf8","#34d399","#fbbf24","#f87171","#a78bfa","#ec4899"];
  const mStr=year+"-"+String(month).padStart(2,"0");
  // Objectif élèves du mois = somme des cours lesson prévus (non annulés, non vacances)
  const objMois=useMemo(()=>events.filter(e=>e.date.startsWith(mStr)&&e.type==="lesson"&&e.price>0&&e.status!=="Annulé"&&e.status!=="Vacances").reduce((a,e)=>a+e.price,0),[events,mStr]);
  const encMois=useMemo(()=>events.filter(e=>e.date.startsWith(mStr)&&e.type==="lesson"&&e.paid).reduce((a,e)=>a+e.price,0),[events,mStr]);
  const encPct=objMois>0?Math.min(Math.round(encMois/objMois*100),100):encMois>0?100:0;
  return(<>
    <input value={q} onChange={e=>setQ(e.target.value)} placeholder="🔍 Rechercher un élève..." style={{width:"100%",background:T.inp,border:"1px solid "+T.inpB,borderRadius:14,padding:"11px 14px",color:T.text,fontSize:13,outline:"none",marginBottom:12,fontFamily:"DM Sans,sans-serif"}}/>
    {/* Barre objectif élèves */}
    <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 16px",marginBottom:14}}>
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-end",marginBottom:10}}>
        <div>
          <div style={{fontSize:10,fontWeight:700,letterSpacing:"0.6px",textTransform:"uppercase",color:T.sec,marginBottom:4}}>🎵 Objectif cours ce mois</div>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:30,lineHeight:1,color:encMois>=objMois&&objMois>0?T.gold:T.green}}>{encMois}<span style={{fontSize:14,color:T.muted}}>€</span></div>
          <div style={{fontSize:10,color:T.faint,marginTop:2}}>encaissé</div>
        </div>
        <div style={{textAlign:"right"}}>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:22,color:T.muted,lineHeight:1}}>{objMois}<span style={{fontSize:11}}>€</span></div>
          <div style={{fontSize:9,color:T.faint,marginTop:2}}>objectif du mois</div>
        </div>
      </div>
      <div style={{height:6,background:T.prog,borderRadius:3,overflow:"hidden",marginBottom:5}}>
        <div style={{height:"100%",borderRadius:3,background:encMois>=objMois&&objMois>0?"linear-gradient(90deg,#fbbf24,#f59e0b)":G,width:encPct+"%",transition:"width .5s"}}/>
      </div>
      <div style={{display:"flex",justifyContent:"space-between",fontSize:9,color:T.faint}}>
        <span>{encPct}%</span>
        <span style={{color:encMois>=objMois&&objMois>0?T.gold:T.faint}}>{encMois>=objMois&&objMois>0?"🏆 Objectif atteint !":objMois>encMois?(objMois-encMois)+"€ restants":""}</span>
      </div>
    </div>
    {list.map(s=>{
      const se=events.filter(e=>e.sid===s.id);
      const sem=se.filter(e=>e.date.startsWith(mStr));
      // Ce mois : dû = cours non annulés/vacances avec prix
      const dueMois=sem.filter(e=>e.price>0&&e.status!=="Annulé"&&e.status!=="Vacances").reduce((a,e)=>a+e.price,0);
      const paidMois=sem.filter(e=>e.paid).reduce((a,e)=>a+e.price,0);
      const resteMois=Math.max(0,dueMois-paidMois);
      // Depuis toujours
      const paidTotal=se.filter(e=>e.paid).reduce((a,e)=>a+e.price,0);
      const cancels=sem.filter(e=>e.status==="Annulé").length;
      const rat=se.filter(e=>e.status==="À Rattraper").length;
      const last=[...se].sort((a,b)=>b.date.localeCompare(a.date))[0];
      const ini=s.name.split(" ").map(w=>w[0]).join("").slice(0,2).toUpperCase();
      const ac=avC[s.id.charCodeAt(1)%6];
      return(<div key={s.id} onClick={()=>onFiche(s)} style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 16px",marginBottom:10,cursor:"pointer"}}>
        <div style={{display:"flex",alignItems:"center",gap:13,marginBottom:10}}>
          <div style={{width:44,height:44,borderRadius:"50%",background:"linear-gradient(135deg,"+ac+","+ac+"77)",display:"flex",alignItems:"center",justifyContent:"center",fontFamily:"DM Serif Display,serif",fontSize:17,color:"white",flexShrink:0,boxShadow:"0 4px 12px "+ac+"33"}}>{ini}</div>
          <div style={{flex:1,minWidth:0}}>
            <div style={{fontSize:14,fontWeight:600,color:T.text,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{s.name}</div>
            <div style={{fontSize:11,color:T.muted,marginTop:2}}>{s.pay||"—"} · {last?fd(last.date):"Aucun cours"}</div>
          </div>
          {/* Résumé paiement mois */}
          <div style={{textAlign:"right",flexShrink:0}}>
            {resteMois>0
              ?<><div style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.gold}}>{resteMois}€ dû</div><div style={{fontSize:10,color:T.green}}>{paidMois>0?paidMois+"€ payé":""}</div></>
              :<><div style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.green}}>✓ {paidMois}€</div><div style={{fontSize:10,color:T.faint}}>réglé ce mois</div></>
            }
          </div>
        </div>
        {/* Barre mini du mois */}
        {dueMois>0&&<div style={{height:3,background:T.prog,borderRadius:2,overflow:"hidden",marginBottom:9}}>
          <div style={{height:"100%",borderRadius:2,background:paidMois>=dueMois?"linear-gradient(90deg,#fbbf24,#f59e0b)":G,width:Math.min(dueMois>0?Math.round(paidMois/dueMois*100):0,100)+"%"}}/>
        </div>}
        <div style={{display:"flex",gap:7,flexWrap:"wrap",alignItems:"center"}}>
          <span style={{fontSize:10,padding:"3px 9px",borderRadius:20,background:T.hi,color:T.accent,border:"1px solid "+T.border}}>{se.filter(e=>e.type==="lesson").length} cours · {paidTotal}€ total</span>
          {cancels>0&&<span style={{fontSize:10,padding:"3px 9px",borderRadius:20,background:"rgba(248,113,113,0.1)",color:T.red,border:"1px solid rgba(248,113,113,0.2)"}}>{cancels} annulé{cancels>1?"s":""}</span>}
          {rat>0&&<span style={{fontSize:10,padding:"3px 9px",borderRadius:20,background:"rgba(251,191,36,0.1)",color:T.gold,border:"1px solid rgba(251,191,36,0.2)"}}>{rat} rattrap.</span>}
        </div>
      </div>);
    })}
  </>);
}

function StudentFiche({student,events,month,year,onClose,onSave,onEditEv}){
  const [s,setS]=useState({...student});
  const [ficheTab,setFicheTab]=useState("mois");
  const [stuObj,setStuObj]=useState(500);
  const [editStuObj,setEditStuObj]=useState(false);
  const [tmpStuObj,setTmpStuObj]=useState("");
  const saveStuObj=()=>{const v=Number(tmpStuObj);if(v>0)setStuObj(v);setEditStuObj(false);};
  const se=useMemo(()=>events.filter(e=>e.sid===student.id).sort((a,b)=>b.date.localeCompare(a.date)),[events,student.id]);
  const mStr=year+"-"+String(month).padStart(2,"0");
  const seMonth=useMemo(()=>se.filter(e=>e.date.startsWith(mStr)),[se,mStr]);
  const mDue=seMonth.filter(e=>e.status!=="Annulé"&&e.status!=="Vacances"&&e.price>0).reduce((a,e)=>a+e.price,0);
  const mPaid=seMonth.filter(e=>e.paid).reduce((a,e)=>a+e.price,0);
  const seHisto=useMemo(()=>se.filter(e=>e.date>="2026-03-01"),[se]);
  const totalDue=seHisto.filter(e=>e.status!=="Annulé"&&e.status!=="Vacances"&&e.price>0).reduce((a,e)=>a+e.price,0);
  const totalPaid=seHisto.filter(e=>e.paid).reduce((a,e)=>a+e.price,0);
  const totalCours=seHisto.filter(e=>e.type==="lesson").length;
  const ini=student.name.split(" ").map(w=>w[0]).join("").slice(0,2).toUpperCase();
  const ac=["#818cf8","#34d399","#fbbf24","#f87171","#a78bfa","#ec4899"][student.id.charCodeAt(1)%6];
  const mapsUrl="https://www.google.com/maps/search/?api=1&query="+encodeURIComponent(s.address||"");
  return(<Overlay onClose={onClose} z={70}><Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <span style={{fontFamily:"DM Serif Display,serif",fontSize:19,color:T.text}}>Fiche élève</span>
      <div style={{display:"flex",gap:8}}><Btn onClick={()=>onSave(s)} style={{padding:"7px 14px",fontSize:12}}>Enregistrer</Btn><CBtn onClose={onClose}/></div>
    </div></SHdr>
    <SBody>
      <div style={{display:"flex",alignItems:"center",gap:14,marginBottom:16,padding:14,background:T.card,border:"1px solid "+T.border,borderRadius:18}}>
        <div style={{width:54,height:54,borderRadius:"50%",background:"linear-gradient(135deg,"+ac+","+ac+"77)",display:"flex",alignItems:"center",justifyContent:"center",fontFamily:"DM Serif Display,serif",fontSize:20,color:"white",flexShrink:0,boxShadow:"0 0 20px "+ac+"44"}}>{ini}</div>
        <div style={{flex:1}}>
          <input style={{width:"100%",fontFamily:"DM Serif Display,serif",fontSize:18,padding:"4px 0",background:"transparent",border:"none",borderBottom:"1px solid "+T.border,color:T.text,outline:"none",marginBottom:6}} value={s.name} onChange={e=>setS({...s,name:e.target.value})}/>
          <div style={{fontSize:11,color:T.muted}}>{totalPaid}€ encaissés · {se.filter(e=>e.type==="lesson").length} cours</div>
        </div>
      </div>
      <div style={{display:"flex",borderBottom:"1px solid "+T.border,marginBottom:12}}>
        {[["mois","📅 Ce mois"],["histo","📈 Depuis mars"]].map(([k,l])=>(
          <div key={k} onClick={()=>setFicheTab(k)} style={{flex:1,padding:"9px 0",textAlign:"center",fontSize:11,fontWeight:700,cursor:"pointer",color:ficheTab===k?T.accent:T.muted,borderBottom:"2px solid "+(ficheTab===k?T.accent:"transparent"),marginBottom:-1}}>{l}</div>
        ))}
      </div>
      {ficheTab==="mois"&&<div style={{marginBottom:14}}>
        {/* 3 infos clés */}
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:9,marginBottom:12}}>
          {[
            {l:"Dû ce mois",v:mDue+"€",c:T.text,i:"📋",s:"cours non annulés"},
            {l:"Payé ce mois",v:mPaid+"€",c:T.green,i:"✅",s:mPaid>=mDue&&mDue>0?"réglé 🎉":"reçu"},
            {l:"Reste à payer",v:Math.max(0,mDue-mPaid)+"€",c:mDue>mPaid?T.gold:T.green,i:mDue>mPaid?"⌛":"✓",s:mDue>mPaid?"en attente":"tout reçu"},
          ].map(k=>(
            <div key={k.l} style={{background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"11px 10px",textAlign:"center"}}>
              <div style={{fontSize:15,marginBottom:3}}>{k.i}</div>
              <div style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:k.c,lineHeight:1}}>{k.v}</div>
              <div style={{fontSize:9,color:T.sec,marginTop:3,textTransform:"uppercase",letterSpacing:"0.4px"}}>{k.l}</div>
              <div style={{fontSize:9,color:T.faint,marginTop:2}}>{k.s}</div>
            </div>
          ))}
        </div>
        {/* Barre progression du mois */}
        {mDue>0&&<div style={{background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"12px 14px",marginBottom:10}}>
          <div style={{height:6,background:T.prog,borderRadius:3,overflow:"hidden",marginBottom:6}}>
            <div style={{height:"100%",borderRadius:3,background:mPaid>=mDue?"linear-gradient(90deg,#fbbf24,#f59e0b)":G,width:Math.min(mDue>0?Math.round(mPaid/mDue*100):0,100)+"%",transition:"width .5s"}}/>
          </div>
          <div style={{display:"flex",justifyContent:"space-between",fontSize:10,color:T.faint}}>
            <span>{mDue>0?Math.round(mPaid/mDue*100):0}% réglé</span>
            <span style={{color:mPaid>=mDue?T.gold:T.faint}}>{mPaid>=mDue?"🏆 Tout réglé !":(mDue-mPaid)+"€ à recevoir"}</span>
          </div>
        </div>}
        <div style={{fontSize:11,color:T.faint,textAlign:"center",padding:"4px 0 8px"}}>Total depuis toujours : <strong style={{color:T.green}}>{totalPaid}€</strong> encaissés sur <strong style={{color:T.text}}>{totalDue}€</strong> facturés</div>
      </div>}
      {ficheTab==="histo"&&<div style={{marginBottom:14}}>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9,marginBottom:10}}>
          {[{l:"Total facturé",v:totalDue+"€",c:T.text,i:"📋"},{l:"Encaissé",v:totalPaid+"€",c:T.green,i:"✅"},{l:"En attente",v:Math.max(0,totalDue-totalPaid)+"€",c:totalDue>totalPaid?T.gold:T.green,i:"⏳"},{l:"Nb cours",v:String(totalCours),c:T.accent,i:"📚"}].map(k=>(
            <div key={k.l} style={{background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"11px 12px"}}>
              <div style={{fontSize:15,marginBottom:4}}>{k.i}</div>
              <div style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:k.c,lineHeight:1}}>{k.v}</div>
              <div style={{fontSize:9,color:T.faint,marginTop:3,textTransform:"uppercase",letterSpacing:"0.4px"}}>{k.l}</div>
            </div>
          ))}
        </div>
        <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"14px 16px"}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:10}}>
            <span style={{fontSize:11,fontWeight:700,color:T.text}}>🎯 Objectif annuel</span>
            {editStuObj
              ?<div style={{display:"flex",gap:6,alignItems:"center"}}>
                <input type="number" value={tmpStuObj} onChange={e=>setTmpStuObj(e.target.value)} style={{width:70,background:T.inp,border:"1px solid "+T.inpB,borderRadius:8,padding:"4px 8px",color:T.text,fontSize:12,outline:"none",fontFamily:"DM Sans,sans-serif"}}/>
                <button onClick={saveStuObj} style={{background:G,border:"none",borderRadius:7,padding:"4px 10px",color:"white",fontSize:11,fontWeight:700,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>OK</button>
              </div>
              :<span onClick={()=>{setTmpStuObj(String(stuObj));setEditStuObj(true);}} style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.muted,cursor:"pointer"}}>{stuObj}€ <span style={{fontSize:10}}>✏️</span></span>
            }
          </div>
          <div style={{height:7,background:T.prog,borderRadius:4,overflow:"hidden",marginBottom:6}}>
            <div style={{height:"100%",borderRadius:4,background:totalPaid>=stuObj?"linear-gradient(90deg,#fbbf24,#f59e0b)":G,width:Math.min(stuObj>0?Math.round(totalPaid/stuObj*100):0,100)+"%",transition:"width .6s"}}/>
          </div>
          <div style={{display:"flex",justifyContent:"space-between",fontSize:10}}>
            <span style={{color:T.green,fontWeight:600}}>{totalPaid}€ encaissés</span>
            <span style={{color:totalPaid>=stuObj?T.gold:T.faint,fontWeight:totalPaid>=stuObj?700:400}}>{totalPaid>=stuObj?"🏆 Objectif atteint !":(stuObj-totalPaid)+"€ restants"}</span>
          </div>
        </div>
        <div style={{fontSize:10,color:T.faint,textAlign:"center",paddingTop:8}}>Depuis mars 2026</div>
      </div>}
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="📞 Téléphone"/><Inp value={s.phone||""} onChange={e=>setS({...s,phone:e.target.value})} placeholder="06..."/></div>
        <div><FLbl text="✉️ Email"/><Inp value={s.email||""} onChange={e=>setS({...s,email:e.target.value})} placeholder="email@..."/></div>
      </div>
      <FLbl text="📍 Adresse"/>
      <div style={{position:"relative",marginBottom:10}}>
        <Inp value={s.address||""} onChange={e=>setS({...s,address:e.target.value})} style={{marginBottom:0,paddingRight:85}}/>
        <a href={mapsUrl} target="_blank" style={{position:"absolute",right:8,top:"50%",transform:"translateY(-50%)",background:G,borderRadius:8,padding:"4px 10px",fontSize:10,fontWeight:600,color:"white",textDecoration:"none"}}>Maps →</a>
      </div>
      <FLbl text="💳 Mode paiement"/><Sel value={s.pay||""} onChange={e=>setS({...s,pay:e.target.value})}><option value="">— Choisir —</option>{PMODES.map(m=><option key={m} value={m}>{m}</option>)}</Sel>
      <FLbl text="🎵 Répertoire & morceaux favoris"/>
      <textarea value={s.repertoire||""} onChange={e=>setS({...s,repertoire:e.target.value})} placeholder="Ex: Madame X a adoré l'impro sur Chopin..." style={{width:"100%",background:T.inp,border:"1px solid "+T.inpB,borderRadius:12,padding:"11px 13px",color:T.text,fontSize:13,outline:"none",minHeight:70,resize:"vertical",marginBottom:10,fontFamily:"DM Sans,sans-serif"}}/>
      <FLbl text="📝 Notes"/>
      <textarea value={s.notes||""} onChange={e=>setS({...s,notes:e.target.value})} style={{width:"100%",background:T.inp,border:"1px solid "+T.inpB,borderRadius:12,padding:"11px 13px",color:T.text,fontSize:13,outline:"none",minHeight:56,resize:"vertical",marginBottom:14,fontFamily:"DM Sans,sans-serif"}}/>
      <SLbl text="Historique" mt={4} mb={10}/>
      {se.map(ev=>{const qs=questStatus(ev);const tc=TC[ev.type]||TC.lesson;return(<div key={ev.id} onClick={()=>onEditEv&&onEditEv({...ev})} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"9px 12px",background:tc.color+"0C",border:"1px solid "+tc.color+"30",borderRadius:12,marginBottom:7,cursor:"pointer"}}>
        <div style={{display:"flex",alignItems:"center",gap:8}}>
          <span style={{fontSize:14}}>{tc.icon}</span>
          <div>
            <div style={{fontSize:12,fontWeight:500,color:T.text}}>{fd(ev.date)} · {ev.time}</div>
            <span style={{fontSize:9,padding:"1px 6px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b,marginTop:3,display:"inline-block"}}>{qs.ico} {qs.label}</span>
          </div>
        </div>
        <div style={{display:"flex",alignItems:"center",gap:10}}>
          <div style={{textAlign:"right"}}>
            <div style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:ev.paid?T.green:T.text}}>{ev.price>0?ev.price+"€":"—"}</div>
            <div style={{fontSize:9,color:ev.paid?T.green:T.gold}}>{ev.paid?"✓":"⌛"}</div>
          </div>
          <span style={{fontSize:11,color:T.faint}}>✏️</span>
        </div>
      </div>);})}
    </SBody>
  </Sheet></Overlay>);
}

function StatsTab({events,monthEvs,stats,pct,month,year,setPanel,setEditing,onMonthSelect}){
  const [exp,setExp]=useState(null);
  const [sector,setSector]=useState("all");
  const fs=ds=>{if(!ds)return"—";const d=new Date(ds+"T12:00");return d.getDate()+" "+MFR[d.getMonth()].slice(0,3);};
  const map={};
  monthEvs.forEach(e=>{if(e.status==="Annulé"||e.status==="Vacances")return;if(!map[e.title])map[e.title]={prev:0,enc:0,evs:[],gig:e.type==="gig"};map[e.title].prev+=e.price;map[e.title].enc+=e.paid?e.price:0;map[e.title].evs.push(e);});
  const entries=Object.entries(map).filter(([,d])=>d.prev>0).sort((a,b)=>b[1].prev-a[1].prev);
  const hist=useMemo(()=>{
    const h=[];
    // Toujours commencer à mars 2026
    const startM=3,startY=2026;
    // Aller jusqu'à mois courant + 3
    let endM=month+3,endY=year;
    while(endM>12){endM-=12;endY++;}
    let m=startM,y=startY;
    while(y<endY||(y===endY&&m<=endM)){
      const me=events.filter(e=>{const d=new Date(e.date);return d.getMonth()+1===m&&d.getFullYear()===y;});
      const by={ehpad:0,wedding:0,resto:0,lesson:0};
      let pv=0,enc=0;
      me.forEach(e=>{if(e.status==="Vacances"||e.status==="Annulé")return;pv+=e.price;if(e.paid)enc+=e.price;const t=(e.title||"").toLowerCase();if(t.includes("ehpad")||t.includes("réside"))by.ehpad+=e.price;else if(t.includes("mariage")||t.includes("wedding"))by.wedding+=e.price;else if(t.includes("resto")||t.includes("restaurant")||t.includes("belem")||t.includes("jazz"))by.resto+=e.price;else if(e.type==="lesson")by.lesson+=e.price;});
      const isCur=m===month&&y===year;
      const isFuture=(y>year)||(y===year&&m>month);
      h.push({lbl:MFR[m-1].slice(0,3),m,y,pv,enc,cur:isCur,future:isFuture,by});
      m++;if(m>12){m=1;y++;}
    }
    return h;
  },[events,month,year]);
  const maxH=Math.max(...hist.map(h=>h.pv),1);
  // pct local = enc / prev (même logique que page Quêtes), indépendant de autoObjectif
  const localPct=stats.prev>0?Math.min(Math.round(stats.enc/stats.prev*100),999):stats.enc>0?100:0;
  const isOk=localPct>=100;
  const sC={ehpad:T.green,wedding:T.red,resto:T.gold,lesson:T.accent};
  const sL={ehpad:"EHPAD",wedding:"Mariages",resto:"Resto",lesson:"Cours"};
  return(<>
    <SLbl text="Fortune du Guerrier" mb={12}/>
    <div onClick={()=>setPanel("enc")} style={{background:T.card,border:"1px solid "+(isOk?"rgba(251,191,36,0.3)":T.border),borderRadius:22,padding:20,marginBottom:14,cursor:"pointer",position:"relative",overflow:"hidden",animation:isOk?"goldGlow 2s infinite":undefined}}>
      {isOk&&<div style={{position:"absolute",top:0,left:0,right:0,height:3,background:GG}}/>}
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"flex-start",marginBottom:14}}>
        <div>
          <SLbl text="Encaissé ce mois" mb={5}/>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:42,lineHeight:1,color:isOk?T.gold:T.green}}>{stats.enc}€</div>
          <div style={{fontSize:11,color:T.green,marginTop:4}}>✓ {localPct}% encaissé</div>
        </div>
        <div style={{textAlign:"right"}}>
          <div style={{fontSize:10,color:T.muted}}>Prévu</div>
          <div style={{fontFamily:"DM Serif Display,serif",fontSize:22,color:T.muted}}>{stats.prev}€</div>
          {stats.ecart>0&&<div style={{fontSize:10,color:T.gold,marginTop:4}}>{stats.ecart}€ à venir</div>}
        </div>
      </div>
      <div style={{height:7,background:T.prog,borderRadius:4,overflow:"hidden"}}>
        <div style={{height:"100%",borderRadius:4,background:isOk?GG:"linear-gradient(90deg,"+T.green+","+T.accent+")",width:Math.min(localPct,100)+"%",transition:"width .6s ease"}}/>
      </div>
    </div>
    <div style={{display:"flex",gap:6,marginBottom:14,overflowX:"auto",paddingBottom:2}}>
      {[["all","Tout"],["ehpad","EHPAD"],["wedding","Mariages"],["resto","Resto"],["lesson","Cours"]].map(([k,l])=>(
        <div key={k} onClick={()=>setSector(k)} style={{padding:"5px 12px",borderRadius:20,fontSize:11,fontWeight:500,cursor:"pointer",background:sector===k?G:T.card,color:sector===k?"white":T.muted,border:"1px solid "+(sector===k?"transparent":T.border),whiteSpace:"nowrap",flexShrink:0}}>{l}</div>
      ))}
    </div>
    <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 14px 12px",marginBottom:14}}>
      <SLbl text="Historique depuis mars 2026" mb={14}/>
      <div style={{overflowX:"auto",paddingBottom:4}}>
        <div style={{display:"flex",alignItems:"flex-end",gap:4,height:120,minWidth:hist.length*40}}>
          {hist.map((d,i)=>{
            const bh=Math.max((d.pv/maxH)*80,d.pv>0?4:0);
            const fh=d.pv>0?Math.max((d.enc/d.pv)*bh,0):0;
            const sv=sector!=="all"?d.by[sector]||0:null;
            const sh=sv!==null&&d.pv>0?Math.max((sv/d.pv)*bh,0):null;
            const showAmt=!d.future&&d.enc>0;
            const showPrev=d.future&&d.pv>0;
            return(
              <div key={i} onClick={()=>!d.future&&onMonthSelect(d.m,d.y)} style={{display:"flex",flexDirection:"column",alignItems:"center",width:36,flexShrink:0,gap:3,cursor:d.future?"default":"pointer"}}>
                {/* Montant affiché au dessus */}
                <div style={{height:22,display:"flex",alignItems:"flex-end",justifyContent:"center",width:"100%"}}>
                  {showAmt&&<span style={{fontSize:d.cur?9:8,color:d.cur?T.accent:T.muted,fontWeight:d.cur?700:500,whiteSpace:"nowrap"}}>{d.enc}€</span>}
                  {showPrev&&<span style={{fontSize:7,color:T.faint,whiteSpace:"nowrap"}}>{d.pv}€</span>}
                </div>
                {/* Barre */}
                <div style={{height:80,display:"flex",flexDirection:"column",justifyContent:"flex-end",width:"100%",position:"relative"}}>
                  {d.pv>0&&<div style={{width:"100%",height:bh,background:d.future?"transparent":d.cur?"rgba(99,102,241,0.22)":"rgba(128,128,128,0.1)",borderRadius:"5px 5px 0 0",border:d.future?"1px dashed rgba(148,163,184,0.3)":d.cur?"1px solid rgba(99,102,241,0.35)":"none",boxShadow:d.cur?"0 0 10px rgba(99,102,241,0.15)":"none"}}/>}
                  {!d.future&&fh>0&&<div style={{position:"absolute",bottom:0,left:0,right:0,height:fh,background:d.cur?G:"rgba(99,102,241,0.5)",borderRadius:"5px 5px 0 0"}}/>}
                  {sh!==null&&!d.future&&sh>0&&<div style={{position:"absolute",bottom:0,left:"15%",right:"15%",height:sh,background:sC[sector]||T.accent,borderRadius:"5px 5px 0 0",opacity:.85}}/>}
                </div>
                {/* Label mois */}
                <span style={{fontSize:8,color:d.cur?T.accent:d.future?"rgba(148,163,184,0.4)":T.faint,fontWeight:d.cur?700:400,marginTop:2}}>{d.lbl}</span>
              </div>
            );
          })}
        </div>
      </div>
      <div style={{display:"flex",gap:10,marginTop:8,fontSize:9,color:T.faint}}>
        <span style={{display:"flex",alignItems:"center",gap:4}}><span style={{width:10,height:10,borderRadius:2,background:G,display:"inline-block"}}/> Encaissé</span>
        <span style={{display:"flex",alignItems:"center",gap:4}}><span style={{width:10,height:10,borderRadius:2,background:"rgba(128,128,128,0.15)",border:"1px dashed rgba(148,163,184,0.4)",display:"inline-block"}}/> À venir</span>
        {sector!=="all"&&<span style={{display:"flex",alignItems:"center",gap:4}}><span style={{width:10,height:10,borderRadius:2,background:sC[sector],display:"inline-block"}}/>{sL[sector]}</span>}
      </div>
    </div>
    {/* ── Répartition par moyen de paiement ── */}
    {(()=>{
      const pmMap={};
      monthEvs.filter(e=>e.paid&&e.price>0).forEach(e=>{
        const k=e.pm||"Non renseigné";
        if(!pmMap[k])pmMap[k]=0;
        pmMap[k]+=e.price;
      });
      const pmEntries=Object.entries(pmMap).sort((a,b)=>b[1]-a[1]);
      const total=pmEntries.reduce((a,[,v])=>a+v,0);
      if(pmEntries.length===0) return null;
      const PM_COLORS={"Virement":"#3b82f6","CESU":"#10b981","CESU+":"#34d399","Espèces":"#f97316","Chèque":"#a78bfa","Envoyé non reçu":"#f87171","Autre":"#94a3b8","Non renseigné":"#64748b"};
      // Donut SVG
      const R=52,r=32,cx=64,cy=64;
      let angle=-Math.PI/2;
      const slices=pmEntries.map(([k,v])=>{
        const frac=v/total;
        const a1=angle,a2=angle+frac*2*Math.PI;
        angle=a2;
        const x1=cx+R*Math.cos(a1),y1=cy+R*Math.sin(a1);
        const x2=cx+R*Math.cos(a2),y2=cy+R*Math.sin(a2);
        const xi1=cx+r*Math.cos(a1),yi1=cy+r*Math.sin(a1);
        const xi2=cx+r*Math.cos(a2),yi2=cy+r*Math.sin(a2);
        const large=frac>0.5?1:0;
        const d=`M${x1},${y1} A${R},${R} 0 ${large},1 ${x2},${y2} L${xi2},${yi2} A${r},${r} 0 ${large},0 ${xi1},${yi1} Z`;
        return{k,v,frac,d,col:PM_COLORS[k]||"#818cf8"};
      });
      return(
        <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 16px",marginBottom:14}}>
          <SLbl text="Moyens de paiement" mb={12}/>
          <div style={{display:"flex",alignItems:"center",gap:16}}>
            {/* Donut */}
            <div style={{flexShrink:0,position:"relative",width:128,height:128}}>
              <svg width={128} height={128} viewBox="0 0 128 128">
                {slices.map((s,i)=>(
                  <path key={i} d={s.d} fill={s.col} opacity={0.9}/>
                ))}
                {/* Cercle central */}
                <circle cx={cx} cy={cy} r={r-1} fill={T.sheet}/>
                <text x={cx} y={cy-6} textAnchor="middle" fontSize={11} fontWeight={700} fill={T.text} fontFamily="DM Sans,sans-serif">{total}€</text>
                <text x={cx} y={cy+8} textAnchor="middle" fontSize={8} fill={T.muted} fontFamily="DM Sans,sans-serif">encaissé</text>
              </svg>
            </div>
            {/* Légende */}
            <div style={{flex:1,display:"flex",flexDirection:"column",gap:7}}>
              {slices.map(s=>(
                <div key={s.k} style={{display:"flex",alignItems:"center",justifyContent:"space-between",gap:8}}>
                  <div style={{display:"flex",alignItems:"center",gap:7,minWidth:0}}>
                    <span style={{width:9,height:9,borderRadius:3,background:s.col,flexShrink:0,display:"inline-block"}}/>
                    <span style={{fontSize:11,color:T.text,fontWeight:500,overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap"}}>{s.k}</span>
                  </div>
                  <div style={{textAlign:"right",flexShrink:0}}>
                    <span style={{fontFamily:"DM Serif Display,serif",fontSize:14,color:T.text}}>{s.v}€</span>
                    <span style={{fontSize:9,color:T.faint,marginLeft:4}}>{Math.round(s.frac*100)}%</span>
                  </div>
                </div>
              ))}
            </div>
          </div>
        </div>
      );
    })()}
    <SLbl text="Détail par client" mb={10}/>
    {entries.map(([name,data])=>{const full=data.enc>=data.prev&&data.prev>0,open=exp===name;const dot=data.gig?T.gold:full?T.green:T.red;const bp=data.prev>0?Math.round((data.enc/data.prev)*100):0;return(
      <div key={name} style={{background:T.card,border:"1px solid "+(open?T.accent+"44":T.border),borderRadius:18,overflow:"hidden",marginBottom:10}}>
        <div onClick={()=>setExp(open?null:name)} style={{padding:"14px 16px",display:"flex",justifyContent:"space-between",alignItems:"center",cursor:"pointer"}}>
          <div style={{display:"flex",alignItems:"center",gap:11}}>
            <div style={{width:9,height:9,borderRadius:"50%",background:dot,boxShadow:"0 0 8px "+dot+"66"}}/>
            <div><div style={{fontSize:13,fontWeight:600,color:T.text}}>{name}</div><div style={{fontSize:9,color:T.faint,marginTop:1}}>{data.evs.length} session{data.evs.length>1?"s":""}</div></div>
          </div>
          <div style={{display:"flex",alignItems:"center",gap:10}}>
            <div style={{textAlign:"right"}}>
              <div style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:full?T.gold:T.text}}>{data.enc}€</div>
              <div style={{fontSize:9,color:T.faint}}>sur {data.prev}€</div>
            </div>
            <span style={{fontSize:14,color:T.faint,transform:open?"rotate(90deg)":"none",transition:"transform .2s"}}>›</span>
          </div>
        </div>
        <div style={{height:2,background:T.prog}}><div style={{height:"100%",background:dot,width:Math.min(bp,100)+"%",transition:"width .4s"}}/></div>
        {open&&<div style={{padding:"10px 16px 14px"}}>
          {data.evs.map(ev=>{const qs=questStatus(ev);const tc=TC[ev.type]||TC.lesson;return(<div key={ev.id} onClick={()=>setEditing({...ev})} style={{display:"flex",justifyContent:"space-between",alignItems:"center",padding:"8px 10px",borderRadius:11,marginBottom:6,background:tc.color+"10",border:"1px solid "+tc.color+"30",cursor:"pointer"}}>
            <div style={{display:"flex",alignItems:"center",gap:8}}>
              <span style={{fontSize:14}}>{tc.icon}</span>
              <span style={{fontSize:10,color:T.muted,width:50}}>{fs(ev.date)}</span>
              <span style={{fontSize:9,padding:"1px 6px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b}}>{qs.ico} {qs.label}</span>
            </div>
            <div style={{display:"flex",alignItems:"center",gap:6}}>
              <span style={{fontFamily:"DM Serif Display,serif",fontSize:14,color:ev.paid?T.green:T.text}}>{ev.price}€</span>
              <span>{ev.paid?"✅":"⌛"}</span>
            </div>
          </div>);})}
          {!full&&<div style={{marginTop:6,padding:"8px 10px",background:"rgba(251,191,36,0.07)",border:"1px solid rgba(251,191,36,0.15)",borderRadius:10,display:"flex",justifyContent:"space-between"}}>
            <span style={{fontSize:11,color:T.gold}}>Butin à encaisser</span>
            <span style={{fontFamily:"DM Serif Display,serif",fontSize:15,color:T.gold}}>{data.prev-data.enc}€</span>
          </div>}
        </div>}
      </div>);})}
  </>);
}

function AdminTab({bioLink,setBioLink,events,recs,students,setEvents,setRecs,setStudents,setMailToast}){
  const [aname,setAname]=useState("Pacôme");
  const exportData=()=>{const d={v:2,date:new Date().toISOString(),events,recs,students};const b=new Blob([JSON.stringify(d,null,2)],{type:"application/json"});const u=URL.createObjectURL(b);const a=document.createElement("a");a.href=u;a.download="piano-backup-"+new Date().toISOString().slice(0,10)+".json";a.click();URL.revokeObjectURL(u);};
  const importData=(file)=>{const r=new FileReader();r.onload=e=>{try{const d=JSON.parse(e.target.result);if(d.events){setEvents(d.events);localStorage.setItem("pm_events",JSON.stringify(d.events));}if(d.recs){setRecs(d.recs);localStorage.setItem("pm_recs",JSON.stringify(d.recs));}if(d.students){setStudents(d.students);localStorage.setItem("pm_students",JSON.stringify(d.students));}alert("✓ Données importées !");}catch(err){alert("Erreur : fichier invalide");}};r.readAsText(file);};
  const [techOk,setTechOk]=useState(false);
  const [copied,setCopied]=useState(null);
  const [mails,setMails]=useState(MAILS0);
  const [editMail,setEditMail]=useState(null);
  const [doc,setDoc]=useState({client:"",date:"",time:"",desc:"Prestation musicale — Piano solo",price:"",type:"Facture",duree:"1h"});
  const [docDone,setDocDone]=useState(false);
  const [docNum,setDocNum]=useState("");
  const fillBio=t=>t.replace(/\{BIO\}/g,bioLink);
  const genDoc=()=>{
    const num=(doc.type==="Facture"?"FAC":"DEV")+"-"+Date.now().toString().slice(-6);
    setDocNum(num);
    const ds=doc.date?new Date(doc.date+"T12:00").toLocaleDateString("fr-FR"):"-";
    const html=`<!DOCTYPE html><html lang="fr"><head><meta charset="UTF-8"/><title>${doc.type} ${num}</title><link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display&family=DM+Sans:wght@400;500;600&display=swap" rel="stylesheet"/><style>*{box-sizing:border-box;margin:0;padding:0;}body{font-family:DM Sans,sans-serif;padding:48px 40px;max-width:680px;margin:0 auto;color:#1a1a2e;}.top{display:flex;justify-content:space-between;padding-bottom:28px;border-bottom:2px solid #1a1a2e;margin-bottom:36px;}.brand{font-family:DM Serif Display,serif;font-size:30px;}.badge{display:inline-block;padding:5px 16px;border:2px solid #1a1a2e;border-radius:30px;font-size:11px;font-weight:700;letter-spacing:1.5px;text-transform:uppercase;}.grid{display:grid;grid-template-columns:1fr 1fr;gap:24px;margin-bottom:36px;}.block label{font-size:10px;font-weight:700;letter-spacing:0.8px;text-transform:uppercase;color:#999;display:block;margin-bottom:6px;}.block p{font-size:14px;line-height:1.6;}table{width:100%;border-collapse:collapse;margin-bottom:24px;}thead tr{background:#1a1a2e;}th{padding:12px 10px;font-size:11px;font-weight:600;text-transform:uppercase;color:white;text-align:left;}th:last-child{text-align:right;}td{padding:14px 10px;border-bottom:1px solid #f0f0f0;font-size:14px;}td:last-child{text-align:right;font-weight:600;}.tot{border-top:2px solid #1a1a2e;padding-top:20px;display:flex;justify-content:space-between;align-items:center;margin-top:4px;}.tl{font-family:DM Serif Display,serif;font-size:20px;}.tv{font-family:DM Serif Display,serif;font-size:32px;}.rib{margin-top:24px;background:#f8f8f8;border-radius:12px;padding:18px 20px;font-size:13px;line-height:2.2;border-left:4px solid #1a1a2e;}.rib strong{display:inline-block;min-width:70px;color:#1a1a2e;}.clause{margin-top:14px;font-size:11px;color:#c00;border:1px solid #f0c0c0;padding:10px 14px;border-radius:8px;background:#fff8f8;}.foot{margin-top:32px;font-size:11px;color:#aaa;display:flex;justify-content:space-between;border-top:1px solid #eee;padding-top:16px;}</style></head><body><div class="top"><div><div class="brand">🎹 ${COMPANY.artist}</div><div style="font-size:12px;color:#666;margin-top:6px;">Musicien — Pianiste professionnel</div><div style="font-size:12px;color:#666;">${COMPANY.phone}</div></div><div style="text-align:right"><span class="badge">${doc.type}</span><div style="font-size:12px;color:#666;margin-top:6px;">${num}</div><div style="font-size:12px;color:#666;">${new Date().toLocaleDateString("fr-FR")}</div></div></div><div class="grid"><div class="block"><label>Pour</label><p><strong>${doc.client||"Client"}</strong></p></div><div class="block"><label>Date de prestation</label><p>${ds}${doc.time?" à "+doc.time:""}</p></div>${doc.duree?'<div class="block"><label>Durée</label><p>'+doc.duree+"</p></div>":""}<div class="block"><label>Référence</label><p>${num}</p></div></div><table><thead><tr><th>Description</th><th style="text-align:right">Montant</th></tr></thead><tbody><tr><td>${doc.desc||"Prestation musicale"}</td><td style="text-align:right">${Number(doc.price||0).toFixed(2)} €</td></tr></tbody></table><div class="tot"><span class="tl">Total ${doc.type==="Facture"?"à régler":""}</span><span class="tv">${Number(doc.price||0).toFixed(2)} €</span></div>${doc.type==="Facture"?'<div class="rib"><strong>IBAN :</strong> '+COMPANY.iban+'<br/><strong>BIC :</strong> '+COMPANY.bic+'<br/><strong>Titulaire :</strong> '+COMPANY.name+'<br/><strong>SIRET :</strong> '+COMPANY.siret+'</div><div class="clause">⚠️ Paiement au maximum 1 mois après prestation.</div>':""}<div class="foot"><span>Merci pour votre confiance !</span><span>SIRET ${COMPANY.siret} · ${COMPANY.name}</span></div></body></html>`;
    const b=new Blob([html],{type:"text/html"});const u=URL.createObjectURL(b);const a=document.createElement("a");a.href=u;a.download=doc.type+"_"+num+".html";a.click();URL.revokeObjectURL(u);
    setDocDone(true);
  };
  const sendDocMail=()=>{
    const sub=doc.type+" "+docNum+" — "+COMPANY.artist;
    const body="Bonjour "+doc.client+",\n\nVeuillez trouver ci-joint ma "+doc.type.toLowerCase()+" n°"+docNum+" d'un montant de "+doc.price+"€.\n\nCordialement,\n"+COMPANY.artist+"\n"+COMPANY.phone+"\n\n"+bioLink;
    safeMail(sub,body,setMailToast);
  };
  const sendTech=()=>{
    const body="Bonjour,\n\nPour ma prestation de demain, j'aurai besoin :\n• Une prise de courant à proximité\n• Un espace d'environ 2m² pour mon piano\n\nMerci !\n"+aname+"\n"+COMPANY.phone+"\n\n"+bioLink;
    safeMail("Fiche technique — "+aname,body,setMailToast);
    setTechOk(true);setTimeout(()=>setTechOk(false),3000);
  };
  const copyMail=m=>{navigator.clipboard.writeText("Sujet : "+m.sub+"\n\n"+fillBio(m.body)).then(()=>{setCopied(m.id);setTimeout(()=>setCopied(null),2500);});};
  const openMail=m=>{safeMail(m.sub,fillBio(m.body),setMailToast);};
  return(<>
    <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 16px",marginBottom:14}}>
      <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:12}}>
        <div style={{width:32,height:32,borderRadius:10,background:G,display:"flex",alignItems:"center",justifyContent:"center",fontSize:15}}>🎹</div>
        <span style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:T.text}}>Identité artiste</span>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="Prénom"/><Inp value={aname} onChange={e=>setAname(e.target.value)} style={{marginBottom:0}}/></div>
        <div><FLbl text="Lien bio / Instagram"/><Inp value={bioLink} onChange={e=>setBioLink(e.target.value)} style={{marginBottom:0}}/></div>
      </div>
      <div style={{fontSize:10,color:T.faint,marginTop:8}}>🔗 Ce lien s'insère automatiquement dans tous vos mails.</div>
    </div>
    <div style={{background:"rgba(99,102,241,0.06)",border:"1px solid rgba(99,102,241,0.2)",borderRadius:18,padding:"14px 16px",marginBottom:14}}>
      <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:8}}>
        <div style={{display:"flex",alignItems:"center",gap:9}}>
          <span style={{fontSize:18}}>📋</span>
          <div><div style={{fontSize:14,fontWeight:600,color:T.text}}>Fiche technique</div><div style={{fontSize:11,color:T.muted}}>Prise de courant + 2m²</div></div>
        </div>
        <button onClick={sendTech} style={{background:techOk?"rgba(52,211,153,0.2)":G,border:"1px solid "+(techOk?"rgba(52,211,153,0.4)":"transparent"),borderRadius:12,padding:"9px 14px",color:techOk?T.green:"white",fontSize:12,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif",transition:"all .3s"}}>{techOk?"✓ Envoyé !":"✉️ Envoyer"}</button>
      </div>
      <div style={{background:T.card,borderRadius:11,padding:"10px 12px",fontSize:11,color:T.muted,lineHeight:1.7,fontStyle:"italic"}}>
        "Bonjour, pour ma prestation de demain, j'aurai besoin d'une prise de courant à proximité et d'un espace de 2m² pour mon piano. Merci ! <strong style={{color:T.text,fontStyle:"normal"}}>{aname}</strong>"
      </div>
    </div>
    <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:18,padding:"14px 16px",marginBottom:14}}>
      <div style={{display:"flex",alignItems:"center",gap:9,marginBottom:14}}>
        <span style={{fontSize:18}}>📄</span>
        <span style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:T.text}}>Facture / Devis Prestige</span>
        <div style={{marginLeft:"auto",display:"flex",background:T.calE,borderRadius:10,padding:3,gap:2}}>
          {["Facture","Devis"].map(tp=>(
            <div key={tp} onClick={()=>{setDoc({...doc,type:tp});setDocDone(false);}} style={{padding:"5px 11px",borderRadius:8,fontSize:11,fontWeight:600,cursor:"pointer",background:doc.type===tp?G:"transparent",color:doc.type===tp?"white":T.muted,transition:"all .2s"}}>{tp}</div>
          ))}
        </div>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="Client"/><Inp value={doc.client} onChange={e=>setDoc({...doc,client:e.target.value})} placeholder="Nom client" style={{marginBottom:0}}/></div>
        <div><FLbl text="Montant €"/><Inp type="number" value={doc.price} onChange={e=>setDoc({...doc,price:e.target.value})} placeholder="150" style={{marginBottom:0}}/></div>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:9,marginTop:9}}>
        <div><FLbl text="Date"/><Inp type="date" value={doc.date} onChange={e=>setDoc({...doc,date:e.target.value})} style={{marginBottom:0}}/></div>
        <div><FLbl text="Heure"/><Inp type="time" value={doc.time} onChange={e=>setDoc({...doc,time:e.target.value})} style={{marginBottom:0}}/></div>
        <div><FLbl text="Durée"/><Inp value={doc.duree} onChange={e=>setDoc({...doc,duree:e.target.value})} placeholder="1h30" style={{marginBottom:0}}/></div>
      </div>
      <div style={{marginTop:9}}><FLbl text="Description"/><Inp value={doc.desc} onChange={e=>setDoc({...doc,desc:e.target.value})} style={{marginBottom:0}}/></div>
      <div style={{background:"rgba(251,191,36,0.06)",border:"1px solid rgba(251,191,36,0.15)",borderRadius:11,padding:"10px 12px",marginTop:10,fontSize:10,color:T.faint,lineHeight:1.8}}>
        💳 IBAN : {COMPANY.iban}<br/>BIC : {COMPANY.bic} · SIRET : {COMPANY.siret}
      </div>
      <div style={{display:"flex",gap:9,marginTop:12}}>
        <Btn onClick={genDoc} style={{flex:1,padding:12,fontSize:13}}>⬇️ Générer & Télécharger</Btn>
        {docDone&&<button onClick={sendDocMail} style={{background:"rgba(52,211,153,0.15)",border:"1px solid rgba(52,211,153,0.3)",borderRadius:12,padding:"12px 14px",color:T.green,fontSize:13,fontWeight:600,cursor:"pointer",whiteSpace:"nowrap",fontFamily:"DM Sans,sans-serif"}}>✉️ Envoyer</button>}
      </div>
      {docDone&&<div style={{marginTop:10,padding:"8px 12px",background:"rgba(52,211,153,0.08)",border:"1px solid rgba(52,211,153,0.2)",borderRadius:10,fontSize:11,color:T.green}}>✓ {doc.type} {docNum} générée</div>}
    </div>
    <SLbl text="📧 Bibliothèque de mails" mb={12}/>
    {mails.map((m,mi)=>(
      <div key={m.id} style={{background:T.card,border:"1px solid "+T.border,borderRadius:16,padding:"14px 16px",marginBottom:11}}>
        <div style={{display:"flex",alignItems:"center",gap:10,marginBottom:10}}>
          <span style={{fontSize:22}}>{m.ico}</span>
          <div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:T.text}}>{m.label}</div><div style={{fontSize:10,color:T.muted,marginTop:1}}>Sujet : {m.sub}</div></div>
          <button onClick={()=>setEditMail(mi)} style={{background:"none",border:"1px solid "+T.border,borderRadius:8,padding:"4px 10px",color:T.muted,fontSize:11,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>✏️</button>
        </div>
        <div style={{background:T.calE,borderRadius:10,padding:"10px 12px",fontSize:11,color:T.muted,lineHeight:1.6,marginBottom:11,maxHeight:64,overflow:"hidden"}}>
          {fillBio(m.body).slice(0,200)}...
        </div>
        <div style={{display:"flex",gap:8}}>
          <button onClick={()=>copyMail(m)} style={{flex:1,background:copied===m.id?"rgba(52,211,153,0.15)":T.calE,border:"1px solid "+(copied===m.id?"rgba(52,211,153,0.3)":T.border),borderRadius:10,padding:"9px",color:copied===m.id?T.green:T.muted,fontSize:12,fontWeight:500,cursor:"pointer",transition:"all .2s",fontFamily:"DM Sans,sans-serif"}}>{copied===m.id?"✓ Copié !":"📋 Copier"}</button>
          <button onClick={()=>openMail(m)} style={{flex:1,background:G,border:"none",borderRadius:10,padding:"9px",color:"white",fontSize:12,fontWeight:500,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>✉️ Ouvrir dans Mail</button>
        </div>
      </div>
    ))}
    {editMail!==null&&(
      <Overlay onClose={()=>setEditMail(null)} z={80}>
        <Sheet full>
          <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
            <span style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.text}}>Modifier le modèle</span>
            <div style={{display:"flex",gap:8}}><Btn onClick={()=>setEditMail(null)} style={{padding:"7px 14px",fontSize:12}}>OK</Btn><CBtn onClose={()=>setEditMail(null)}/></div>
          </div></SHdr>
          <SBody>
            <FLbl text="Sujet"/>
            <Inp value={mails[editMail].sub} onChange={e=>{const m=[...mails];m[editMail]={...m[editMail],sub:e.target.value};setMails(m);}}/>
            <FLbl text="Corps du mail"/>
            <textarea value={mails[editMail].body} onChange={e=>{const m=[...mails];m[editMail]={...m[editMail],body:e.target.value};setMails(m);}} style={{width:"100%",background:T.inp,border:"1px solid "+T.inpB,borderRadius:12,padding:"11px 13px",color:T.text,fontSize:12,outline:"none",minHeight:300,resize:"vertical",fontFamily:"DM Sans,sans-serif",lineHeight:1.7}}/>
            <div style={{fontSize:10,color:T.faint,marginTop:6}}>Utilisez {"{"+"BIO"+"}"} pour insérer votre lien automatiquement.</div>
          </SBody>
        </Sheet>
      </Overlay>
    )}
    <div style={{display:"flex",gap:9,marginTop:8}}>
      <button onClick={exportData} style={{flex:1,background:T.card,border:"1px solid "+T.border,borderRadius:12,padding:"11px",color:T.muted,fontSize:12,fontWeight:500,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>💾 Exporter données</button>
      <label style={{flex:1,background:T.card,border:"1px solid "+T.border,borderRadius:12,padding:"11px",color:T.muted,fontSize:12,fontWeight:500,cursor:"pointer",textAlign:"center",fontFamily:"DM Sans,sans-serif"}}>
        📂 Importer<input type="file" accept=".json" onChange={e=>e.target.files[0]&&importData(e.target.files[0])} style={{display:"none"}}/>
      </label>
    </div>
  </>);
}

function EditSheet({ev,setEv,onSave,onDel,onClose}){
  const qs=questStatus(ev);
  const [confirmDel,setConfirmDel]=useState(false);
  // Validation : si paid=true, status doit être "Fait" et pm doit être rempli
  const errors=[];
  if(!ev.title.trim()) errors.push("Le titre est obligatoire");
  if(!ev.date) errors.push("La date est obligatoire");
  if(ev.paid && !ev.pm) errors.push("Mode de paiement requis quand paiement reçu");
  if(ev.paid && ev.status!=="Fait") errors.push('Le statut doit être Fait si le paiement est reçu');
  if(ev.paid && ev.price<=0) errors.push("Le prix doit être > 0 si paiement reçu");
  const canSave=errors.length===0;
  return(<Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <div>
        <span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>{ev.title||"Événement"}</span>
        <div style={{display:"flex",alignItems:"center",gap:6,marginTop:4}}>
          <span style={{fontSize:10,padding:"2px 8px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b,fontWeight:600}}>{qs.ico} {qs.label}</span>
          {ev.price>0&&<span style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:ev.paid?T.green:T.gold}}>{ev.price}€</span>}
        </div>
      </div>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <SBody>
      <FLbl text="Titre"/><Inp value={ev.title} onChange={e=>setEv({...ev,title:e.target.value})} placeholder="Nom..." style={!ev.title.trim()?{borderColor:"rgba(248,113,113,0.5)"}:{}}/>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="Date"/><Inp type="date" value={ev.date} onChange={e=>setEv({...ev,date:e.target.value})}/></div>
        <div><FLbl text="Heure"/><Inp type="time" value={ev.time} onChange={e=>setEv({...ev,time:e.target.value})}/></div>
      </div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="Prix €"/><Inp type="number" value={ev.price} onChange={e=>setEv({...ev,price:Number(e.target.value)})} min={0}/></div>
        <div><FLbl text="Type"/><Sel value={ev.type} onChange={e=>setEv({...ev,type:e.target.value})}>{Object.entries(TC).map(([k,v])=><option key={k} value={k}>{v.label}</option>)}</Sel></div>
      </div>
      <FLbl text="Statut"/>
      <Sel value={ev.status} onChange={e=>setEv({...ev,status:e.target.value,paid:e.target.value!=="Fait"?false:ev.paid})}>{["Prévu","Fait","À Rattraper","Annulé","Vacances"].map(s=><option key={s} value={s}>{s}</option>)}</Sel>
      {ev.status==="À Rattraper"&&(
        <div style={{background:"rgba(251,191,36,0.08)",border:"1px solid rgba(251,191,36,0.2)",borderRadius:14,padding:"12px 14px",marginBottom:14}}>
          <div style={{fontSize:12,fontWeight:600,color:T.gold,marginBottom:10}}>🔄 Timeline du décalage</div>
          <div style={{display:"flex",alignItems:"center",gap:8}}>
            <div style={{flex:1}}><div style={{fontSize:10,color:T.red,marginBottom:4}}>❌ Date annulée</div><Inp type="date" value={ev.origDate||""} onChange={e=>setEv({...ev,origDate:e.target.value})} style={{marginBottom:0,borderColor:"rgba(248,113,113,0.3)"}}/></div>
            <div style={{fontSize:18,color:"rgba(251,191,36,0.5)",paddingTop:14}}>→</div>
            <div style={{flex:1}}><div style={{fontSize:10,color:T.gold,marginBottom:4}}>✅ Nouveau rdv</div><Inp type="date" value={ev.date} onChange={e=>setEv({...ev,date:e.target.value})} style={{marginBottom:0,borderColor:"rgba(251,191,36,0.3)"}}/></div>
          </div>
        </div>
      )}
      <div style={{height:1,background:T.border,margin:"4px 0 14px"}}/>
      <label style={{display:"flex",alignItems:"center",justifyContent:"space-between",padding:"12px 14px",background:T.card,border:"1px solid "+(ev.paid&&!ev.pm?"rgba(248,113,113,0.4)":T.border),borderRadius:13,cursor:"pointer",marginBottom:10}}>
        <span style={{fontSize:13,fontWeight:500,color:T.text,display:"flex",alignItems:"center",gap:9}}>
          <input type="checkbox" checked={ev.paid} onChange={e=>setEv({...ev,paid:e.target.checked,status:e.target.checked?"Fait":ev.status})} style={{width:17,height:17,accentColor:T.accent}}/>Paiement reçu
        </span>
        {ev.price>0&&<span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:ev.paid?T.green:T.muted}}>{ev.price}€</span>}
      </label>
      <div style={{display:"flex",alignItems:"center",gap:4,marginBottom:5}}>
        <span style={{fontSize:10,fontWeight:700,letterSpacing:"0.6px",textTransform:"uppercase",color:T.sec}}>Mode paiement</span>
        {ev.paid&&<span style={{color:T.red,fontSize:11,fontWeight:700}}>*</span>}
      </div>
      <Sel value={ev.pm||""} onChange={e=>setEv({...ev,pm:e.target.value||null})} style={ev.paid&&!ev.pm?{borderColor:"rgba(248,113,113,0.5)",background:"rgba(248,113,113,0.05)"}:{}}><option value="">— Choisir —</option>{PMODES.map(m=><option key={m} value={m}>{m}</option>)}</Sel>
      {/* Erreurs de validation */}
      {errors.length>0&&<div style={{background:"rgba(248,113,113,0.08)",border:"1px solid rgba(248,113,113,0.25)",borderRadius:11,padding:"10px 12px",marginBottom:10}}>
        {errors.map((e,i)=><div key={i} style={{fontSize:11,color:T.red,display:"flex",alignItems:"center",gap:6,marginBottom:i<errors.length-1?4:0}}><span>⚠️</span>{e}</div>)}
      </div>}
      <div style={{display:"flex",justifyContent:"space-between",alignItems:"center",marginTop:4,gap:10}}>
        {!confirmDel
          ?<button onClick={()=>setConfirmDel(true)} style={{background:"rgba(248,113,113,0.08)",border:"1px solid rgba(248,113,113,0.2)",borderRadius:11,padding:"10px 14px",color:T.red,fontSize:12,fontWeight:500,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>🗑 Supprimer</button>
          :<div style={{display:"flex",gap:7,alignItems:"center"}}>
            <span style={{fontSize:11,color:T.red}}>Confirmer ?</span>
            <button onClick={onDel} style={{background:"rgba(248,113,113,0.15)",border:"1px solid rgba(248,113,113,0.4)",borderRadius:9,padding:"7px 12px",color:T.red,fontSize:11,fontWeight:700,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>Oui, supprimer</button>
            <button onClick={()=>setConfirmDel(false)} style={{background:T.card,border:"1px solid "+T.border,borderRadius:9,padding:"7px 12px",color:T.muted,fontSize:11,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>Annuler</button>
          </div>
        }
        <div style={{position:"relative"}}>
          <Btn onClick={canSave?onSave:undefined} style={{padding:"12px 22px",fontSize:14,opacity:canSave?1:0.45,cursor:canSave?"pointer":"not-allowed"}} title={canSave?"":"Remplis tous les champs requis"}>Enregistrer</Btn>
        </div>
      </div>
    </SBody>
  </Sheet>);
}

function AddSheet({onAdd,onClose}){
  const [ev,setEv]=useState({title:"",date:"2026-03-01",time:"10:00",price:50,status:"Prévu",paid:false,pm:null,type:"lesson",origDate:null,sid:null});
  const addErrors=[];
  if(!ev.title.trim()) addErrors.push("Titre obligatoire");
  if(!ev.date) addErrors.push("Date obligatoire");
  if(ev.paid && !ev.pm) addErrors.push("Mode de paiement requis");
  if(ev.paid && Number(ev.price)<=0) addErrors.push("Prix > 0 requis si paiement reçu");
  const add=()=>{if(addErrors.length>0)return;onAdd({...ev,price:Number(ev.price)});onClose();};
  return(<Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>Nouvel événement</span>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <SBody>
      <SLbl text="Type" mb={9}/>
      <div style={{display:"grid",gridTemplateColumns:"repeat(3,1fr)",gap:8,marginBottom:14}}>
        {Object.entries(TC).map(([k,cfg])=>(
          <div key={k} onClick={()=>setEv({...ev,type:k})} style={{padding:"10px 5px",borderRadius:13,border:"1.5px solid "+(ev.type===k?T.accent:T.border),background:ev.type===k?T.accent+"15":T.card,textAlign:"center",cursor:"pointer",fontSize:10,color:ev.type===k?T.text:T.muted,transition:"all .15s"}}>
            <div style={{fontSize:18,marginBottom:4}}>{cfg.icon}</div>{cfg.label}
          </div>
        ))}
      </div>
      <FLbl text="Titre"/><Inp value={ev.title} onChange={e=>setEv({...ev,title:e.target.value})} placeholder={ev.type==="task"?"Ex: Travailler Chopin, Appeler client...":"Nom de l'élève..."}/>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
        <div><FLbl text="Date"/><Inp type="date" value={ev.date} onChange={e=>setEv({...ev,date:e.target.value})}/></div>
        <div><FLbl text="Heure"/><Inp type="time" value={ev.time} onChange={e=>setEv({...ev,time:e.target.value})}/></div>
      </div>
      {ev.type==="task"
        ?<div style={{padding:"10px 14px",background:"rgba(148,163,184,0.06)",border:"1px dashed rgba(148,163,184,0.25)",borderRadius:10,marginBottom:16,fontSize:11,color:T.muted}}>📌 Simple rappel — pas de suivi de paiement.</div>
        :<><div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
          <div><FLbl text="Prix €"/><Inp type="number" value={ev.price} onChange={e=>setEv({...ev,price:e.target.value})} min={0}/></div>
          <div><FLbl text="Statut"/><Sel value={ev.status} onChange={e=>setEv({...ev,status:e.target.value})}>{["Prévu","Fait","À Rattraper","Annulé","Vacances"].map(s=><option key={s} value={s}>{s}</option>)}</Sel></div>
        </div>
        <label style={{display:"flex",alignItems:"center",gap:9,padding:"12px 14px",background:T.card,border:"1px solid "+T.border,borderRadius:13,cursor:"pointer",marginBottom:12}}>
          <input type="checkbox" checked={ev.paid} onChange={e=>setEv({...ev,paid:e.target.checked})} style={{width:17,height:17,accentColor:T.accent}}/>
          <span style={{fontSize:13,fontWeight:500,color:T.text}}>Déjà payé</span>
        </label></>
      }
      {addErrors.length>0&&<div style={{background:"rgba(248,113,113,0.08)",border:"1px solid rgba(248,113,113,0.25)",borderRadius:11,padding:"10px 12px",marginBottom:10}}>
        {addErrors.map((e,i)=><div key={i} style={{fontSize:11,color:T.red,display:"flex",alignItems:"center",gap:6,marginBottom:i<addErrors.length-1?4:0}}><span>⚠️</span>{e}</div>)}
      </div>}
      <Btn onClick={add} style={{width:"100%",padding:14,fontSize:14,opacity:addErrors.length===0?1:0.45,cursor:addErrors.length===0?"pointer":"not-allowed"}}>Ajouter</Btn>
    </SBody>
  </Sheet>);
}

function DetailPanel({pid,events,monthEvs,stats,pct,month,year,onClose,onEdit}){
  const bodies={
    ca:()=>{const items=monthEvs.filter(e=>e.status!=="Vacances"&&e.status!=="Annulé"&&e.price>0);return(<><div style={{textAlign:"center",padding:"4px 0 20px",borderBottom:"1px solid "+T.border,marginBottom:16}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:46,color:T.accent,lineHeight:1}}>{stats.prev}€</div><div style={{fontSize:12,color:T.muted,marginTop:4}}>{items.length} cours facturables</div></div>{items.map(e=>{const qs=questStatus(e);const tc=TC[e.type]||TC.lesson;return(<div key={e.id} onClick={()=>onEdit({...e})} style={{background:tc.color+"0C",border:"1px solid "+tc.color+"35",borderRadius:14,padding:"11px 13px",cursor:"pointer",display:"flex",alignItems:"center",gap:10,marginBottom:7}}><div style={{width:32,height:32,borderRadius:9,background:tc.color+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:15,flexShrink:0}}>{tc.icon}</div><div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:T.text}}>{e.title}</div><div style={{fontSize:10,color:T.muted}}>{fd(e.date)} · {e.time}</div></div><div style={{textAlign:"right"}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:e.paid?T.green:T.text}}>{e.price}€</div><span style={{fontSize:9,padding:"1px 6px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b}}>{qs.ico} {qs.label}</span></div></div>);})}</>);},
    enc:()=>{const paid=monthEvs.filter(e=>e.paid),unpaid=monthEvs.filter(e=>!e.paid&&e.price>0&&e.status!=="Vacances"&&e.status!=="Annulé");const Row=({e})=>{const tc=TC[e.type]||TC.lesson;const qs=questStatus(e);return(<div onClick={()=>onEdit({...e})} style={{background:tc.color+"0C",border:"1px solid "+tc.color+"35",borderRadius:14,padding:"11px 13px",cursor:"pointer",display:"flex",alignItems:"center",gap:10,marginBottom:7}}><div style={{width:30,height:30,borderRadius:8,background:tc.color+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:14,flexShrink:0}}>{tc.icon}</div><div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:T.text}}>{e.title}</div><div style={{fontSize:10,color:T.muted}}>{fd(e.date)}</div></div><div style={{textAlign:"right"}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:16,color:e.paid?T.green:qs.col}}>{e.price}€{e.paid?" ✓":""}</div><span style={{fontSize:9,padding:"1px 6px",borderRadius:20,background:qs.bg,color:qs.col,border:"1px solid "+qs.b}}>{qs.ico} {qs.label}</span></div></div>);};return(<><div style={{textAlign:"center",padding:"4px 0 20px",borderBottom:"1px solid "+T.border,marginBottom:16}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:46,color:T.green,lineHeight:1}}>{stats.enc}€</div><div style={{fontSize:12,color:T.muted,marginTop:4}}>{stats.prev>0?Math.min(Math.round(stats.enc/stats.prev*100),999):100}% · {paid.length} paiements</div></div>{paid.length>0&&<><SLbl text="✅ Payés" mb={9}/>{paid.map(e=><Row key={e.id} e={e}/>)}</>}{unpaid.length>0&&<><SLbl text="⏳ En attente" mt={12} mb={9}/>{unpaid.map(e=><Row key={e.id} e={e}/>)}</>}</>);},
    due:()=>{const due=monthEvs.filter(e=>!e.paid&&e.price>0&&e.status!=="Vacances"&&e.status!=="Annulé");const byS={};due.forEach(e=>{if(!byS[e.title])byS[e.title]={t:0,items:[]};byS[e.title].t+=e.price;byS[e.title].items.push(e);});return(<><div style={{textAlign:"center",padding:"4px 0 20px",borderBottom:"1px solid "+T.border,marginBottom:16}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:46,color:T.gold,lineHeight:1}}>{stats.ecart}€</div><div style={{fontSize:12,color:T.muted,marginTop:4}}>{due.length} cours non réglés</div></div>{due.length===0?<div style={{textAlign:"center",padding:28,color:T.faint}}>🎉 Tout est réglé !</div>:Object.entries(byS).sort((a,b)=>b[1].t-a[1].t).map(([n,d])=><div key={n} style={{marginBottom:14}}><div style={{display:"flex",justifyContent:"space-between",marginBottom:7}}><span style={{fontSize:13,fontWeight:600,color:T.text}}>{n}</span><span style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.gold}}>{d.t}€</span></div>{d.items.map(e=>{const qs=questStatus(e);return(<div key={e.id} onClick={()=>onEdit({...e})} style={{background:T.card,border:"1px solid "+qs.b,borderRadius:14,padding:"11px 13px",cursor:"pointer",display:"flex",justifyContent:"space-between",marginBottom:7}}><div style={{fontSize:12,color:T.muted}}>{fd(e.date)} · {e.time}</div><span style={{fontFamily:"DM Serif Display,serif",fontSize:15,color:qs.col}}>{e.price}€ {qs.ico}</span></div>);})}</div>)}</>);},
    rat:()=>{const mStr=year+"-"+String(month).padStart(2,"0");const items=events.filter(e=>e.date.startsWith(mStr)&&e.price>0&&(e.status==="À Rattraper"||e.status==="Annulé"));const totalPerdu=items.filter(e=>e.status==="Annulé").reduce((a,e)=>a+e.price,0);return(<><div style={{textAlign:"center",padding:"4px 0 20px",borderBottom:"1px solid "+T.border,marginBottom:16}}><div style={{fontFamily:"DM Serif Display,serif",fontSize:46,color:T.red,lineHeight:1}}>{items.length}</div><div style={{fontSize:12,color:T.muted,marginTop:4}}>{totalPerdu>0?totalPerdu+"€ annulés · ":""}{items.filter(e=>e.status==="À Rattraper").length} rattrap.</div></div>{items.length===0?<div style={{textAlign:"center",padding:24,color:T.faint}}>✅ Aucune perte ce mois</div>:items.map(e=>(
      <div key={e.id} onClick={()=>onEdit(e)} style={{background:T.card,border:"1px solid "+(e.status==="Annulé"?"rgba(248,113,113,0.3)":"rgba(251,191,36,0.3)"),borderRadius:16,padding:14,marginBottom:10,cursor:"pointer"}}>
        <div style={{display:"flex",justifyContent:"space-between",marginBottom:8}}><span style={{fontSize:13,fontWeight:600,color:T.text}}>{e.title}</span><span style={{fontFamily:"DM Serif Display,serif",fontSize:14,color:e.status==="Annulé"?T.red:T.gold}}>{e.price}€ {e.status==="Annulé"?"❌":"🔄"}</span></div>
        <div style={{fontSize:11,color:T.muted}}>{fd(e.date)} · {e.time}</div>
        {e.origDate&&<div style={{fontSize:10,color:T.gold,marginTop:4}}>était le {fd(e.origDate)}</div>}
      </div>
    ))}</>);},
  };
  const titles={ca:"CA Prévu",enc:"Encaissé",due:"Butin en attente",rat:"Pertes & Rattrapages"};
  const icos={ca:"💰",enc:"✅",due:"⏳",rat:"⚠️"};
  if(!bodies[pid]) return null;
  return(<Overlay onClose={onClose} z={60}><Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <div style={{display:"flex",alignItems:"center",gap:10}}><div style={{width:34,height:34,borderRadius:11,background:T.accent+"22",display:"flex",alignItems:"center",justifyContent:"center",fontSize:16}}>{icos[pid]}</div><span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>{titles[pid]}</span></div>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <SBody>{bodies[pid]()}</SBody>
  </Sheet></Overlay>);
}

function AlertsPanel({events,onClose,onEdit,onRelance}){
  const cut=new Date();cut.setDate(cut.getDate()-14);const cs=cut.toISOString().split("T")[0];
  const old=events.filter(e=>!e.paid&&e.price>0&&e.status==="Fait"&&e.date<=cs);
  const byS={};old.forEach(e=>{if(!byS[e.title])byS[e.title]={t:0,c:0,last:""};byS[e.title].t+=e.price;byS[e.title].c++;if(!byS[e.title].last||e.date>byS[e.title].last)byS[e.title].last=e.date;});
  const urgC=ds=>{const d=dAgo(ds);return d>30?T.red:d>14?T.gold:T.green;};
  return(<Overlay onClose={onClose} z={60}><Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <div style={{display:"flex",alignItems:"center",gap:10}}><span style={{fontSize:18}}>🔔</span><span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>Rappels paiement</span></div>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <SBody>
      {Object.keys(byS).length===0?<div style={{textAlign:"center",padding:40}}><div style={{fontSize:36,marginBottom:10}}>🎉</div><div style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>Tout est à jour !</div></div>:
        Object.entries(byS).sort((a,b)=>b[1].t-a[1].t).map(([n,d])=>{
          const c=urgC(d.last);
          const daysLate=dAgo(d.last);
          const relanceTxt=encodeURIComponent("Salut ! Je faisais un point compta, je crois que le virement pour la prestation du "+fd(d.last)+" n'est pas encore passé. Tiens-moi au courant ! Pacôme");
          return(<div key={n} style={{background:T.card,border:"1px solid "+c+"33",borderRadius:14,padding:"13px 15px",marginBottom:10}}>
            <div style={{display:"flex",justifyContent:"space-between",marginBottom:8}}><span style={{fontSize:14,fontWeight:600,color:T.text}}>{n}</span><span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:c}}>{d.t}€</span></div>
            <div style={{display:"flex",gap:8,alignItems:"center",marginBottom:10}}>
              <span style={{fontSize:10,color:T.muted}}>{d.c} cours · dernier : {fd(d.last)}</span>
              <span style={{fontSize:9,padding:"2px 8px",borderRadius:20,background:c+"18",color:c,border:"1px solid "+c+"40",fontWeight:500}}>{daysLate}j de retard</span>
            </div>
            <div style={{display:"flex",gap:8}}>
              <button onClick={()=>window.open("sms:?body="+relanceTxt)} style={{flex:1,background:"rgba(96,165,250,0.12)",border:"1px solid rgba(96,165,250,0.3)",borderRadius:10,padding:"8px",color:T.blue,fontSize:11,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>📱 SMS Relance</button>
              <button onClick={()=>{const body="Bonjour "+n+",\n\nJe faisais un point compta, je crois que le virement pour la prestation du "+fd(d.last)+" n'est pas encore passé.\n\nTiens-moi au courant !\nPacôme";safeMail("Point paiement — "+n,body,null);}} style={{flex:1,background:G,border:"none",borderRadius:10,padding:"8px",color:"white",fontSize:11,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>✉️ Mail Relance</button>
            </div>
          </div>);
        })
      }
    </SBody>
  </Sheet></Overlay>);
}

function RecPanel({recs,setRecs,events,setEvents,month,year,onClose}){
  const [adding,setAdding]=useState(false);
  const [nr,setNr]=useState({title:"",time:"10:00",price:50,type:"lesson",dow:1,active:true});
  const save=()=>{if(!nr.title.trim())return;const id="rc"+_RID++;const r={...nr,id,price:Number(nr.price)};setRecs(p=>[...p,r]);setEvents(p=>[...p,...genRec([r],month,year,p)]);setAdding(false);setNr({title:"",time:"10:00",price:50,type:"lesson",dow:1,active:true});};
  return(<Overlay onClose={onClose} z={60}><Sheet full>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <div style={{display:"flex",alignItems:"center",gap:10}}><span style={{fontSize:18}}>🔁</span><span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>Récurrences</span></div>
      <div style={{display:"flex",gap:8}}><Btn onClick={()=>setAdding(!adding)} style={{padding:"7px 13px",fontSize:12}}>+ Ajouter</Btn><CBtn onClose={onClose}/></div>
    </div></SHdr>
    <SBody>
      {adding&&<div style={{background:T.accent+"0D",border:"1px solid "+T.accent+"33",borderRadius:16,padding:14,marginBottom:16}}>
        <SLbl text="Nouveau cours récurrent" color={T.accent} mb={10}/>
        <Inp value={nr.title} onChange={e=>setNr({...nr,title:e.target.value})} placeholder="Nom élève"/>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
          <div><FLbl text="Jour"/><Sel value={nr.dow} onChange={e=>setNr({...nr,dow:Number(e.target.value)})}>{DFULL.map((d,i)=><option key={i} value={i}>{d}</option>)}</Sel></div>
          <div><FLbl text="Heure"/><Inp type="time" value={nr.time} onChange={e=>setNr({...nr,time:e.target.value})}/></div>
        </div>
        <div style={{display:"grid",gridTemplateColumns:"1fr 1fr",gap:9}}>
          <div><FLbl text="Tarif €"/><Inp type="number" value={nr.price} onChange={e=>setNr({...nr,price:e.target.value})} min={0}/></div>
          <div><FLbl text="Type"/><Sel value={nr.type} onChange={e=>setNr({...nr,type:e.target.value})}>{Object.entries(TC).map(([k,v])=><option key={k} value={k}>{v.label}</option>)}</Sel></div>
        </div>
        <div style={{display:"flex",gap:8}}><Btn onClick={save} style={{flex:1,padding:12,fontSize:13}}>Créer</Btn><button onClick={()=>setAdding(false)} style={{background:T.card,border:"1px solid "+T.border,borderRadius:12,padding:12,color:T.muted,fontSize:13,cursor:"pointer",width:44,fontFamily:"DM Sans,sans-serif"}}>✕</button></div>
      </div>}
      {recs.length===0&&<div style={{textAlign:"center",padding:28,color:T.faint}}>Aucun cours récurrent</div>}
      {recs.map(r=>{const tc=TC[r.type]||TC.lesson;return(
        <div key={r.id} style={{background:T.card,border:"1px solid "+(r.active?T.accent+"33":T.border),borderRadius:15,padding:"13px 15px",marginBottom:10,opacity:r.active?1:0.55}}>
          <div style={{display:"flex",alignItems:"center",gap:11,marginBottom:9}}>
            <div style={{width:9,height:9,borderRadius:"50%",background:tc.color}}/>
            <div style={{flex:1}}><div style={{fontSize:13,fontWeight:600,color:T.text}}>{r.title}</div><div style={{fontSize:10,color:T.muted,marginTop:1}}>{DFULL[r.dow]}s · {r.time} · {r.price>0?r.price+"€":"Gratuit"}</div></div>
            <div style={{display:"flex",gap:6}}>
              <button onClick={()=>setRecs(p=>p.map(x=>x.id===r.id?{...x,active:!x.active}:x))} style={{background:r.active?"rgba(52,211,153,0.15)":T.card,border:"1px solid "+(r.active?"rgba(52,211,153,0.3)":T.border),borderRadius:8,padding:"4px 9px",color:r.active?T.green:T.muted,fontSize:10,fontWeight:500,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>{r.active?"Actif":"Inactif"}</button>
              <button onClick={()=>{if(window.confirm("Supprimer cette récurrence ? Elle n'apparaîtra plus dans les prochains mois."))setRecs(p=>p.filter(x=>x.id!==r.id));}} style={{background:"rgba(248,113,113,0.1)",border:"1px solid rgba(248,113,113,0.2)",borderRadius:8,width:28,height:28,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:T.red,fontSize:14,fontFamily:"DM Sans,sans-serif"}}>×</button>
            </div>
          </div>
          {r.active&&<button onClick={()=>{const g=genRec([r],month,year,events);if(g.length)setEvents(p=>[...p,...g]);}} style={{background:T.accent+"18",border:"1px solid "+T.accent+"33",borderRadius:8,padding:"4px 10px",color:T.accent,fontSize:10,fontWeight:500,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>🔁 Générer {MFR[month-1].slice(0,4)}.</button>}
        </div>);})}
    </SBody>
  </Sheet></Overlay>);
}

function RelanceModal({ev,onClose}){
  const txt="Salut ! Je faisais un point compta, je crois que le virement pour la prestation du "+fd(ev.date)+" n'est pas encore passé. Tiens-moi au courant ! Pacôme";
  const [copied,setCopied]=useState(false);
  const copy=()=>{navigator.clipboard.writeText(txt).then(()=>{setCopied(true);setTimeout(()=>setCopied(false),2000);});};
  return(<Overlay onClose={onClose} z={75}><Sheet>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <span style={{fontFamily:"DM Serif Display,serif",fontSize:18,color:T.text}}>📱 Relance SMS / Mail</span>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <div style={{padding:"8px 20px 32px"}}>
      <div style={{fontSize:12,color:T.muted,marginBottom:8}}>Pour : <strong style={{color:T.text}}>{ev.title}</strong> · {ev.price}€ · {fd(ev.date)}</div>
      <div style={{background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"14px 16px",fontSize:13,color:T.text,lineHeight:1.7,marginBottom:14,fontStyle:"italic"}}>"{txt}"</div>
      <div style={{display:"grid",gridTemplateColumns:"1fr 1fr 1fr",gap:8}}>
        <button onClick={copy} style={{background:copied?"rgba(52,211,153,0.15)":T.card,border:"1px solid "+(copied?"rgba(52,211,153,0.3)":T.border),borderRadius:12,padding:"11px 8px",color:copied?T.green:T.muted,fontSize:11,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif",transition:"all .2s"}}>{copied?"✓ Copié":"📋 Copier"}</button>
        <button onClick={()=>window.open("sms:?body="+encodeURIComponent(txt))} style={{background:"rgba(96,165,250,0.15)",border:"1px solid rgba(96,165,250,0.3)",borderRadius:12,padding:"11px 8px",color:T.blue,fontSize:11,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>📱 SMS</button>
        <button onClick={()=>safeMail("Point paiement — "+ev.title,txt,null)} style={{background:G,border:"none",borderRadius:12,padding:"11px 8px",color:"white",fontSize:11,fontWeight:600,cursor:"pointer",fontFamily:"DM Sans,sans-serif"}}>✉️ Mail</button>
      </div>
    </div>
  </Sheet></Overlay>);
}

function ThemePicker({cur,onChange,onClose}){
  const previews={
    dark:{bg:"#070712",c1:"#6366f1",c2:"#8b5cf6",c3:"#34d399",desc:"Nuit étoilée, sobre et élégant"},
    sky:{bg:"linear-gradient(160deg,#7dd3fc 0%,#bae6fd 50%,#e0f2fe 100%)",c1:"#0284c7",c2:"#fbbf24",c3:"#059669",desc:"Ciel bleu, nuages et soleil"},
    rose:{bg:"linear-gradient(135deg,#fce7f3,#fdf2f8)",c1:"#ec4899",c2:"#f43f5e",c3:"#059669",desc:"Sakura, douceur florale"},
    cyber:{bg:"linear-gradient(135deg,#060612,#0d0d22)",c1:"#a855f7",c2:"#06b6d4",c3:"#f43f5e",desc:"Néons violets et cyan, futuriste"},
    forest:{bg:"linear-gradient(135deg,#052e16,#071c0e)",c1:"#4ade80",c2:"#fbbf24",c3:"#93c5fd",desc:"Sous-bois profond, vert intense"},
    gold:{bg:"linear-gradient(135deg,#0a0800,#130f00)",c1:"#fbbf24",c2:"#f59e0b",c3:"#34d399",desc:"Or sur noir, luxe et prestige"},
  };
  return(<Overlay onClose={onClose} z={80}><Sheet>
    <SHdr><div style={{display:"flex",alignItems:"center",justifyContent:"space-between"}}>
      <span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>🎨 Thème visuel</span>
      <CBtn onClose={onClose}/>
    </div></SHdr>
    <div style={{padding:"0 20px 32px",display:"flex",flexDirection:"column",gap:10}}>
      {Object.entries(THEMES).map(([k,th])=>{
        const isActive=cur===k;
        const pv=previews[k]||{bg:th.bg,c1:th.accent,c2:th.accent2,c3:th.green,desc:""};
        return(<div key={k} onClick={()=>{onChange(k);onClose();}} style={{display:"flex",alignItems:"center",gap:14,padding:"14px 16px",background:isActive?th.accent+"20":T.card,border:"2px solid "+(isActive?th.accent:T.border),borderRadius:18,cursor:"pointer",transition:"all .2s"}}>
          {/* Preview miniature */}
          <div style={{width:64,height:48,borderRadius:12,background:pv.bg,flexShrink:0,overflow:"hidden",position:"relative",boxShadow:"0 4px 12px rgba(0,0,0,0.3)"}}>
            {/* Mini barres de couleur */}
            <div style={{position:"absolute",bottom:6,left:6,right:6,display:"flex",gap:3}}>
              {[pv.c1,pv.c2,pv.c3].map((c,i)=>(
                <div key={i} style={{flex:1,height:i===0?16:i===1?10:13,background:c,borderRadius:3,opacity:0.9}}/>
              ))}
            </div>
            {/* Soleil pour sky */}
            {k==="sky"&&<div style={{position:"absolute",top:4,right:8,width:14,height:14,borderRadius:"50%",background:"#fbbf24",boxShadow:"0 0 8px #fbbf2488"}}/>}
            {/* Nuage pour sky */}
            {k==="sky"&&<div style={{position:"absolute",top:6,left:6,fontSize:12}}>☁️</div>}
            {/* Étoile cyber */}
            {k==="cyber"&&<div style={{position:"absolute",top:4,left:8,fontSize:10}}>⚡</div>}
            {/* Étoiles dark */}
            {k==="dark"&&["12,8","28,14","44,6","8,22"].map((pos,i)=><div key={i} style={{position:"absolute",top:pos.split(",")[1]+"px",left:pos.split(",")[0]+"px",width:2,height:2,borderRadius:"50%",background:"white",opacity:0.7}}/>)}
          </div>
          <div style={{flex:1,minWidth:0}}>
            <div style={{fontSize:14,fontWeight:700,color:T.text,marginBottom:4}}>{th.name}</div>
            <div style={{fontSize:10,color:T.muted,lineHeight:1.4}}>{pv.desc}</div>
          </div>
          {isActive&&<div style={{width:24,height:24,borderRadius:"50%",background:"linear-gradient(135deg,"+th.accent+","+th.accent2+")",display:"flex",alignItems:"center",justifyContent:"center",color:"white",fontSize:13,flexShrink:0}}>✓</div>}
        </div>);
      })}
    </div>
  </Sheet></Overlay>);
}

function App(){
  function lsGet(key,def){try{const v=localStorage.getItem(key);if(v===null)return def;const p=JSON.parse(v);return(Array.isArray(p)&&p.length>0)||(!Array.isArray(p)&&p!==null)?p:def;}catch(e){return def;}}
  function lsSet(key,val){try{localStorage.setItem(key,JSON.stringify(val));return true;}catch(e){return false;}}
  const [events,setEvents]=useState(()=>lsGet("pm_events",EVENTS0));
  const [recs,setRecs]=useState(()=>lsGet("pm_recs",RECS0));
  const [students,setStudents]=useState(()=>lsGet("pm_students",STUDENTS0));
  const [month,setMonth]=useState(()=>{const v=lsGet("pm_month",null);return(v&&Number(v)>=1&&Number(v)<=12)?Number(v):3;});
  const [year,setYear]=useState(()=>Number(lsGet("pm_year",2026)));
  const [tab,setTab]=useState(()=>lsGet("pm_tab","planning"));
  const [editing,setEditing]=useState(null);
  const [adding,setAdding]=useState(false);
  const [panel,setPanel]=useState(null);
  const [alerts,setAlerts]=useState(false);
  const [recPanel,setRecPanel]=useState(false);
  const [fiche,setFiche]=useState(null);
  const [relance,setRelance]=useState(null);
  const [tid,setTid]=useState(()=>lsGet("pm_theme","dark"));
  T=THEMES[tid]||THEMES.dark;
  G=`linear-gradient(135deg,${T.accent},${T.accent2})`;
  const [themeOpen,setThemeOpen]=useState(false);
  const [mailToast,setMailToast]=useState(null);
  React.useEffect(()=>{if(mailToast){const t=setTimeout(()=>setMailToast(null),3500);return()=>clearTimeout(t);}},[mailToast]);
  const [saved,setSaved]=useState(false);
  const isFirst=React.useRef(true);
  const [bioLink,setBioLink]=useState(()=>lsGet("pm_biolink","https://instagram.com/pacome_piano"));

  const monthEvs=useMemo(()=>events.filter(e=>{const d=new Date(e.date);return d.getMonth()+1===month&&d.getFullYear()===year;}),[events,month,year]);
  const stats=useMemo(()=>{let prev=0,enc=0,ecart=0,rat=0,done=0,total=0;monthEvs.forEach(e=>{if(e.status!=="Vacances"&&e.status!=="Annulé")prev+=e.price;if(e.paid)enc+=e.price;if(e.status==="À Rattraper")rat++;if(e.type==="lesson"){total++;if(e.status==="Fait")done++;}});ecart=prev-enc;return{prev,enc,ecart,rat,done,total};},[monthEvs]);
  const autoObjectif=useMemo(()=>{const dim=new Date(year,month,0).getDate();let obj=0;recs.filter(r=>r.active&&r.price>0).forEach(r=>{for(let d=1;d<=dim;d++){if(new Date(year,month-1,d).getDay()===r.dow)obj+=r.price;}});return obj||0;},[recs,month,year]);
  const pct=autoObjectif>0?Math.min(Math.round(stats.prev/autoObjectif*100),999):stats.prev>0?100:0;
  useEffect(()=>{if(isFirst.current){isFirst.current=false;return;}if(lsSet("pm_events",events)){setSaved(true);setTimeout(()=>setSaved(false),2000);}},[events]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_recs",recs);},[recs]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_students",students);},[students]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_month",month);},[month]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_year",year);},[year]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_tab",tab);},[tab]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_theme",tid);},[tid]);
  useEffect(()=>{if(!isFirst.current)lsSet("pm_biolink",bioLink);},[bioLink]);
  const alertCount=useMemo(()=>{const cut=new Date();cut.setDate(cut.getDate()-14);const cs=cut.toISOString().split("T")[0];return events.filter(e=>!e.paid&&e.price>0&&e.status==="Fait"&&e.date<=cs).length;},[events]);
  const sorted=[...monthEvs].sort((a,b)=>a.date.localeCompare(b.date)||a.time.localeCompare(b.time));
  const prevM=()=>{if(month===1){setMonth(12);setYear(y=>y-1);}else setMonth(m=>m-1);};
  const nextM=()=>{if(month===12){setMonth(1);setYear(y=>y+1);}else setMonth(m=>m+1);};
  const saveEv=()=>{setEvents(ev=>ev.map(x=>x.id===editing.id?editing:x));setEditing(null);};
  const delEv=()=>{setEvents(ev=>ev.filter(x=>x.id!==editing.id));setEditing(null);};
  const addEv=ev=>setEvents(p=>[...p,{...ev,id:_ID++}]);

  const TABS=[{id:"planning",ico:"⚔️",lbl:"Quêtes"},{id:"calendar",ico:"📅",lbl:"Calendrier"},{id:"students",ico:"👤",lbl:"Élèves"},{id:"charts",ico:"📊",lbl:"Fortune"},{id:"admin",ico:"🎼",lbl:"Admin"}];

  return(
    <div style={{width:"100%",minHeight:"100vh",height:"100dvh",background:T.outer,display:"flex",flexDirection:"column",overflow:"hidden",position:"relative"}}>
      {/* Nuages animés — thème Ciel uniquement */}
      {tid==="sky"&&<div style={{position:"absolute",inset:0,pointerEvents:"none",zIndex:0,overflow:"hidden"}}>
        {/* Soleil */}
        <div style={{position:"absolute",top:18,right:28,width:52,height:52,borderRadius:"50%",background:"radial-gradient(circle,#fef08a,#fbbf24)",boxShadow:"0 0 40px rgba(251,191,36,0.6), 0 0 80px rgba(251,191,36,0.2)"}}/>
        {/* Rayons soleil */}
        {[0,45,90,135].map(deg=><div key={deg} style={{position:"absolute",top:44,right:54,width:30,height:2,background:"rgba(251,191,36,0.35)",borderRadius:1,transformOrigin:"left center",transform:"rotate("+deg+"deg) translateX(30px)"}}/>)}
        {/* Nuage 1 — gros */}
        <div style={{position:"absolute",top:"8%",left:0,animation:"floatCloud 28s linear infinite",willChange:"transform"}}>
          <div style={{position:"relative",width:130,height:45}}>
            <div style={{position:"absolute",top:15,left:10,width:110,height:30,borderRadius:20,background:"rgba(255,255,255,0.92)",boxShadow:"0 4px 15px rgba(255,255,255,0.4)"}}/>
            <div style={{position:"absolute",top:5,left:30,width:60,height:40,borderRadius:"50%",background:"rgba(255,255,255,0.95)"}}/>
            <div style={{position:"absolute",top:10,left:55,width:45,height:35,borderRadius:"50%",background:"rgba(255,255,255,0.9)"}}/>
          </div>
        </div>
        {/* Nuage 2 — petit */}
        <div style={{position:"absolute",top:"20%",left:0,animation:"floatCloud 40s linear 8s infinite",willChange:"transform"}}>
          <div style={{position:"relative",width:90,height:30}}>
            <div style={{position:"absolute",top:10,left:5,width:80,height:20,borderRadius:15,background:"rgba(255,255,255,0.8)"}}/>
            <div style={{position:"absolute",top:2,left:20,width:40,height:30,borderRadius:"50%",background:"rgba(255,255,255,0.85)"}}/>
          </div>
        </div>
        {/* Nuage 3 — moyen */}
        <div style={{position:"absolute",top:"35%",left:0,animation:"floatCloud 35s linear 18s infinite",willChange:"transform"}}>
          <div style={{position:"relative",width:110,height:38}}>
            <div style={{position:"absolute",top:12,left:8,width:95,height:26,borderRadius:18,background:"rgba(255,255,255,0.75)"}}/>
            <div style={{position:"absolute",top:3,left:25,width:50,height:35,borderRadius:"50%",background:"rgba(255,255,255,0.8)"}}/>
            <div style={{position:"absolute",top:6,left:55,width:38,height:28,borderRadius:"50%",background:"rgba(255,255,255,0.78)"}}/>
          </div>
        </div>
      </div>}
      {/* Grille cyber — thème Cyber */}
      {tid==="cyber"&&<div style={{position:"absolute",inset:0,pointerEvents:"none",zIndex:0,backgroundImage:"linear-gradient(rgba(139,92,246,0.07) 1px,transparent 1px),linear-gradient(90deg,rgba(139,92,246,0.07) 1px,transparent 1px)",backgroundSize:"32px 32px"}}/>}
      {/* Étoiles — thème Sombre */}
      {tid==="dark"&&<div style={{position:"absolute",inset:0,pointerEvents:"none",zIndex:0}}>
        {Array.from({length:28}).map((_,i)=>{
          const x=(i*137.5)%100, y=(i*97.3)%85, s=i%3===0?2.5:i%3===1?1.5:1, op=0.3+((i*73)%10)*0.05;
          return <div key={i} style={{position:"absolute",left:x+"%",top:y+"%",width:s,height:s,borderRadius:"50%",background:"white",opacity:op}}/>;
        })}
      </div>}
      <style>{"@keyframes slideUp{from{transform:translateY(100%)}to{transform:translateY(0)}} @keyframes pulseR{0%,100%{opacity:1}50%{opacity:.3}} @keyframes goldGlow{0%,100%{box-shadow:0 0 8px rgba(251,191,36,0.3)}50%{box-shadow:0 0 24px rgba(251,191,36,0.6)}} @keyframes floatCloud{0%{transform:translateX(-150px) scaleX(1)}50%{transform:translateX(calc(50vw)) scaleX(1.05)}100%{transform:translateX(calc(100vw + 150px)) scaleX(1)}} @keyframes floatCloud2{0%{transform:translateX(-100px)}100%{transform:translateX(calc(100vw + 100px))}} input,select,textarea,button{font-family:DM Sans,sans-serif;} a{text-decoration:none;} *{-webkit-tap-highlight-color:transparent;}"}</style>
      <div style={{flex:1,background:T.phone,display:"flex",flexDirection:"column",position:"relative",overflow:"hidden"}}>
        <div style={{padding:"4px 20px 10px",flexShrink:0,position:"relative",zIndex:2}}>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",marginBottom:10}}>
            <div style={{display:"flex",alignItems:"center",gap:10}}>
              <div style={{width:32,height:32,background:G,borderRadius:11,display:"flex",alignItems:"center",justifyContent:"center",fontSize:16}}>🎹</div>
              <span style={{fontFamily:"DM Serif Display,serif",fontSize:20,color:T.text}}>Pacôme</span>
            </div>
            <div style={{display:"flex",gap:7,alignItems:"center"}}>
              <div onClick={()=>setAlerts(true)} style={{width:34,height:34,border:"1px solid "+T.border,borderRadius:11,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",background:T.card,position:"relative",fontSize:15}}>
                🔔{alertCount>0&&<div style={{position:"absolute",top:-4,right:-4,width:15,height:15,background:T.red,borderRadius:"50%",fontSize:8,fontWeight:700,color:"white",display:"flex",alignItems:"center",justifyContent:"center",border:"2px solid "+T.phone}}>{alertCount}</div>}
              </div>
              <div onClick={()=>setThemeOpen(true)} style={{width:34,height:34,border:"1px solid "+T.border,borderRadius:11,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",background:T.card,fontSize:15}}>🎨</div>
              <div onClick={()=>setAdding(true)} style={{width:34,height:34,background:G,borderRadius:11,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",color:"white",fontSize:20,boxShadow:"0 4px 12px "+T.accent+"44"}}>+</div>
            </div>
          </div>
          <div style={{display:"flex",alignItems:"center",justifyContent:"space-between",background:T.card,border:"1px solid "+T.border,borderRadius:14,padding:"9px 16px",marginBottom:8}}>
            <button onClick={prevM} style={{background:"none",border:"none",color:T.muted,cursor:"pointer",fontSize:20,lineHeight:1}}>‹</button>
            <span style={{fontFamily:"DM Serif Display,serif",fontSize:17,color:T.text}}>{MFR[month-1]} {year}</span>
            <button onClick={nextM} style={{background:"none",border:"none",color:T.muted,cursor:"pointer",fontSize:20,lineHeight:1}}>›</button>
          </div>

        </div>

        <div style={{flex:1,overflowY:"auto",padding:"0 16px 10px",position:"relative",zIndex:2}}>
          {tab==="planning"&&<PlanningTab monthEvs={monthEvs} sorted={sorted} stats={stats} pct={pct} setPanel={setPanel} alertCount={alertCount} setAlerts={setAlerts} setRecPanel={setRecPanel} recs={recs} setEditing={setEditing} onRelance={setRelance}/>}
          {tab==="calendar"&&<CalendarTab events={events} setEvents={setEvents} month={month} year={year} onEdit={setEditing}/>}
          {tab==="students"&&<StudentsTab students={students} events={events} onFiche={setFiche} month={month} year={year}/>}
          {tab==="charts"&&<StatsTab events={events} monthEvs={monthEvs} stats={stats} pct={pct} month={month} year={year} setPanel={setPanel} setEditing={setEditing} onMonthSelect={(m,y)=>{setMonth(m);setYear(y);}}/>}
          {tab==="admin"&&<AdminTab bioLink={bioLink} setBioLink={setBioLink} events={events} recs={recs} students={students} setEvents={setEvents} setRecs={setRecs} setStudents={setStudents} setMailToast={setMailToast}/>}
        </div>

        <nav style={{flexShrink:0,background:T.nav,borderTop:"1px solid "+T.navB,padding:"8px 0 22px",display:"flex",position:"relative",zIndex:2}}>
          {TABS.map(t=>(
            <div key={t.id} onClick={()=>setTab(t.id)} style={{flex:1,display:"flex",flexDirection:"column",alignItems:"center",gap:3,cursor:"pointer",padding:"5px 0"}}>
              <div style={{width:36,height:26,borderRadius:10,background:tab===t.id?T.accent+"22":"transparent",display:"flex",alignItems:"center",justifyContent:"center",fontSize:16,transition:"background .2s"}}>{t.ico}</div>
              <span style={{fontSize:9,fontWeight:500,color:tab===t.id?T.accent:T.muted}}>{t.lbl}</span>
            </div>
          ))}
        </nav>

        {panel&&<DetailPanel pid={panel} events={events} monthEvs={monthEvs} stats={stats} pct={pct} month={month} year={year} onClose={()=>setPanel(null)} onEdit={ev=>{setPanel(null);setTimeout(()=>setEditing({...ev}),50);}}/>}
        {alerts&&<AlertsPanel events={events} onClose={()=>setAlerts(false)} onEdit={ev=>{setAlerts(false);setTimeout(()=>setEditing({...ev}),50);}} onRelance={ev=>{setAlerts(false);setRelance(ev);}}/>}
        {recPanel&&<RecPanel recs={recs} setRecs={setRecs} events={events} setEvents={setEvents} month={month} year={year} onClose={()=>setRecPanel(false)}/>}
        {fiche&&<StudentFiche student={fiche} events={events} month={month} year={year} onClose={()=>setFiche(null)} onSave={s=>{setStudents(p=>p.map(x=>x.id===s.id?s:x));setFiche(null);}} onEditEv={ev=>{setFiche(null);setEditing(ev);}}/>}
        {saved&&<div style={{position:"fixed",top:16,left:"50%",transform:"translateX(-50%)",background:"rgba(52,211,153,0.15)",border:"1px solid rgba(52,211,153,0.3)",borderRadius:20,padding:"6px 16px",fontSize:11,fontWeight:700,color:"#34d399",zIndex:200,pointerEvents:"none"}}>✓ Sauvegardé</div>}
        {themeOpen&&<ThemePicker cur={tid} onChange={setTid} onClose={()=>setThemeOpen(false)}/>}
        <MailToast msg={mailToast}/>
        {relance&&<RelanceModal ev={relance} onClose={()=>setRelance(null)}/>}
        {editing&&<Overlay onClose={()=>setEditing(null)} z={55}><EditSheet ev={editing} setEv={setEditing} onSave={saveEv} onDel={delEv} onClose={()=>setEditing(null)}/></Overlay>}
        {adding&&<Overlay onClose={()=>setAdding(false)} z={55}><AddSheet onAdd={addEv} onClose={()=>setAdding(false)}/></Overlay>}
      </div>
    </div>
  );
}

ReactDOM.createRoot(document.getElementById("root")).render(<App/>);
</script>

<!-- PWA Service Worker -->
<script>
if('serviceWorker' in navigator){
  // Inline service worker as blob
  const swCode = `
const CACHE = 'piano-pro-v1';
const ASSETS = []; // app est un fichier unique, on cache tout ce qui passe

self.addEventListener('install', e => {
  self.skipWaiting();
});

self.addEventListener('activate', e => {
  e.waitUntil(
    caches.keys().then(keys => Promise.all(
      keys.filter(k => k !== CACHE).map(k => caches.delete(k))
    ))
  );
  self.clients.claim();
});

self.addEventListener('fetch', e => {
  // Network first pour les CDN (React, Babel, fonts)
  // Cache first pour le fichier HTML principal
  const url = new URL(e.request.url);
  if(url.hostname === location.hostname || url.protocol === 'file:') {
    e.respondWith(
      caches.open(CACHE).then(cache =>
        cache.match(e.request).then(cached => {
          const fresh = fetch(e.request).then(res => {
            if(res.ok) cache.put(e.request, res.clone());
            return res;
          }).catch(() => cached);
          return cached || fresh;
        })
      )
    );
  }
});
`;
  const blob = new Blob([swCode], {type:'application/javascript'});
  const swUrl = URL.createObjectURL(blob);
  navigator.serviceWorker.register(swUrl, {scope:'/'})
    .then(r => console.log('SW registered'))
    .catch(e => console.log('SW error:', e));
}
</script>

<!-- Manifest inline via JS pour fichier unique -->
<script>
(function(){
  const manifest = {
    name: "Piano Pro — Pacôme",
    short_name: "Piano Pro",
    description: "Gestion cours de piano et prestations",
    start_url: "./",
    display: "standalone",
    background_color: "#0f0e17",
    theme_color: "#0f0e17",
    orientation: "portrait",
    icons: [
      {
        src: "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 192 192'%3E%3Crect width='192' height='192' rx='40' fill='%230f0e17'/%3E%3Ctext x='96' y='130' font-size='100' text-anchor='middle'%3E🎹%3C/text%3E%3C/svg%3E",
        sizes: "192x192",
        type: "image/svg+xml"
      },
      {
        src: "data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 512 512'%3E%3Crect width='512' height='512' rx='100' fill='%230f0e17'/%3E%3Ctext x='256' y='360' font-size='280' text-anchor='middle'%3E🎹%3C/text%3E%3C/svg%3E",
        sizes: "512x512",
        type: "image/svg+xml"
      }
    ]
  };
  const blob = new Blob([JSON.stringify(manifest)], {type:'application/manifest+json'});
  const url = URL.createObjectURL(blob);
  const link = document.createElement('link');
  link.rel = 'manifest';
  link.href = url;
  document.head.appendChild(link);
})();
</script>
</body>
</html>
