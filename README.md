<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CLSI 2025 Finder by Asik</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f7f8fc;
            background-image: linear-gradient(to right, #eef2ff 1px, transparent 1px), linear-gradient(to bottom, #eef2ff 1px, transparent 1px);
            background-size: 40px 40px;
        }
        .header-gradient {
            background: linear-gradient(135deg, #1e3a8a, #4f46e5);
        }
        .search-container {
            position: sticky;
            top: 0;
            background-color: rgba(247, 248, 252, 0.8);
            backdrop-filter: blur(10px);
            z-index: 20;
            padding-bottom: 1rem;
            border-bottom: 1px solid #e5e7eb;
        }
        .accordion-header {
            cursor: pointer;
            transition: background-color 0.2s;
        }
        .accordion-header:hover {
            background-color: #f9fafb;
        }
        .accordion-content {
            max-height: 0;
            overflow: hidden;
            transition: max-height 0.3s ease-out, padding 0.3s ease-out;
        }
        .accordion-content.open {
            max-height: 1000px; /* Adjust as needed */
            padding-top: 1rem;
            padding-bottom: 1rem;
        }
        .icon {
            width: 24px;
            height: 24px;
            margin-right: 0.75rem;
            flex-shrink: 0;
        }
        /* Custom radio button styles */
        .search-mode-label {
            display: inline-block;
            padding: 0.5rem 1rem;
            border: 1px solid #d1d5db;
            border-radius: 9999px;
            cursor: pointer;
            transition: all 0.2s;
            font-weight: 500;
        }
        .search-mode-input:checked + .search-mode-label {
            background-color: #3b82f6;
            color: white;
            border-color: #3b82f6;
            box-shadow: 0 1px 3px rgba(0,0,0,0.1);
        }
    </style>
</head>
<body class="text-gray-800">

    <div class="header-gradient text-white p-8 text-center shadow-lg">
        <h1 class="text-3xl md:text-4xl font-bold tracking-tight">CLSI 2025 Finder</h1>
        <p class="text-blue-200 mt-2 font-medium">by Asik</p>
    </div>

    <div class="container mx-auto p-4 md:p-6 max-w-5xl">

        <!-- Search Section -->
        <div class="search-container py-4">
            <div class="bg-white p-6 rounded-xl shadow-lg border border-gray-200">
                <!-- Search Mode Toggle -->
                <div class="flex justify-center space-x-4 mb-6">
                    <div>
                        <input type="radio" id="mode-breakpoint" name="search-mode" value="breakpoint" class="hidden search-mode-input" checked>
                        <label for="mode-breakpoint" class="search-mode-label">Breakpoint Search</label>
                    </div>
                    <div>
                        <input type="radio" id="mode-intrinsic" name="search-mode" value="intrinsic" class="hidden search-mode-input">
                        <label for="mode-intrinsic" class="search-mode-label">Intrinsic Resistance</label>
                    </div>
                </div>

                <div class="grid grid-cols-1 md:grid-cols-2 gap-4 items-center">
                    <div>
                        <label for="bug-search" class="block text-sm font-medium text-gray-700 mb-1">Organism (Bug)</label>
                        <input type="text" id="bug-search" list="bug-list" placeholder="e.g., Staphylococcus aureus" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 transition">
                        <datalist id="bug-list"></datalist>
                    </div>
                    <div id="drug-search-container">
                        <label for="drug-search" class="block text-sm font-medium text-gray-700 mb-1">Antimicrobial (Drug)</label>
                        <input type="text" id="drug-search" list="drug-list" placeholder="e.g., Vancomycin" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500 transition">
                        <datalist id="drug-list"></datalist>
                    </div>
                </div>
                <div class="mt-4 text-center">
                     <button id="search-btn" class="w-full md:w-auto bg-indigo-600 text-white font-semibold px-8 py-2.5 rounded-lg hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition-all duration-200 shadow-md hover:shadow-lg transform hover:-translate-y-0.5">Search</button>
                </div>
            </div>
        </div>

        <!-- Results Section -->
        <div id="results-container" class="mt-6 space-y-6">
             <div id="initial-message" class="text-center text-gray-500 bg-white p-10 rounded-xl shadow-sm border border-gray-200">
                <svg class="mx-auto h-12 w-12 text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
                    <path vector-effect="non-scaling-stroke" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
                </svg>
                <p class="mt-4 text-lg">Select a search mode above to begin.</p>
                <p class="mt-2 text-sm">Your personalized notes will appear here once you search and save them.</p>
            </div>
        </div>
        
        <!-- User ID display -->
        <div id="user-info" class="fixed bottom-2 right-2 text-xs text-gray-400 p-2 bg-white/80 backdrop-blur-sm rounded-lg shadow">
            User ID: <span id="user-id-display"></span>
        </div>

    </div>

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, onSnapshot, collection, query, where, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- DATA (DEFINITIVE EDITION v3) ---
        const astData = [
            {
                organismGroup: "Enterobacterales",
                aliases: ["enterobacterales", "e. coli", "escherichia coli", "klebsiella", "proteus", "citrobacter", "enterobacter", "serratia", "morganella"],
                data: [
                    {
                        antimicrobial: "Ampicillin",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥17", I: "14-16^", R: "≤13" },
                        mic: { S: "≤8", I: "16^", R: "≥32" },
                        comments: {
                            exam: ["Predicts amoxicillin susceptibility.", "Oral breakpoints are for uncomplicated UTIs due to *E. coli* & *P. mirabilis* only.", "Many Enterobacterales (*Klebsiella*, *Enterobacter*, *Citrobacter*, *Serratia*) are intrinsically resistant."],
                            mechanism: ["Penicillinase-labile penicillin. Resistance is primarily due to β-lactamase production (e.g., TEM-1)."]
                        }
                    },
                    {
                        antimicrobial: "Cefazolin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥23", I: "20-22", R: "≤19" },
                        mic: { S: "≤2", I: "4", R: "≥8" },
                        comments: {
                            exam: ["A 1st-generation cephalosporin. Used as a surrogate for oral cephalosporins for uncomplicated UTIs.", "Not active against ESBL or AmpC producers.", "Breakpoints differ for UTI vs. other infections for *E. coli*, *K. pneumoniae*, *P. mirabilis*."],
                            mechanism: ["Inhibits cell wall synthesis. Resistance via β-lactamases (ESBL, AmpC)."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥23", I: "20-22^", R: "≤19" },
                        mic: { S: "≤1", I: "2^", R: "≥4" },
                        comments: {
                            exam: ["A 3rd-generation cephalosporin. Key agent for many Gram-negative infections.", "Resistance is a primary indicator for ESBL screening.", "<u>WARNING:</u> May appear active but is clinically ineffective for serious infections caused by AmpC-producers like *Enterobacter cloacae* due to risk of derepression during therapy."],
                            mechanism: ["Hydrolyzed by ESBL and AmpC β-lactamases."]
                        }
                    },
                    {
                        antimicrobial: "Cefepime",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥25", SDD: "19-24", R: "≤18" },
                        mic: { S: "≤2", SDD: "4-8", R: "≥16" },
                        comments: {
                            exam: ["A 4th-generation cephalosporin, valued for its stability against many chromosomally-mediated AmpC β-lactamases.", "Often a preferred agent for infections caused by 'SPICE' organisms (*Serratia, Providencia, Indole-positive Proteae, Citrobacter, Enterobacter*).", "The SDD category is critical; it necessitates higher doses (2g IV q8h) to treat organisms with elevated MICs.", "**WARNING:** Cefepime S/SDD results should be suppressed or reported as resistant for isolates that demonstrate carbapenemase production."],
                            mechanism: ["Binds to Penicillin-Binding Proteins (PBPs), inhibiting cell wall synthesis. Forms a stable zwitterion, allowing better penetration through the outer membrane of Gram-negatives.", "Resistance is primarily due to hydrolysis by ESBLs and carbapenemases. Hyperproduction of AmpC combined with porin loss can also confer resistance."]
                        }
                    },
                    {
                        antimicrobial: "Piperacillin-tazobactam",
                        diskContent: "100/10 µg",
                        zoneDiameter: { S: "≥25", SDD: "21-24", R: "≤20" },
                        mic: { S: "≤8/4", SDD: "16/4", R: "≥32/4" },
                        comments: {
                            exam: ["Tazobactam inhibits most common plasmid-mediated β-lactamases (like TEM, SHV) but is **not effective against AmpC or carbapenemases**.", "For isolates with SDD (Susceptible-Dose Dependent) results, **extended infusions** (e.g., 4.5g IV over 3-4 hours q8h) are often necessary to achieve therapeutic targets (%T > MIC).", "Infections caused by AmpC-producing organisms (e.g., *Enterobacter*, *Citrobacter*) may not respond well, even if susceptible in vitro, due to potential for derepression."],
                            mechanism: ["Piperacillin is a ureidopenicillin with broad-spectrum activity. Tazobactam is a β-lactamase inhibitor.", "Resistance arises from: 1) Production of β-lactamases not inhibited by tazobactam (AmpC, carbapenemases, some ESBLs). 2) Hyperproduction of standard β-lactamases. 3) Altered penicillin-binding proteins (PBPs)."]
                        }
                    },
                    {
                        antimicrobial: "Ertapenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥22", I: "19-21^", R: "≤18" },
                        mic: { S: "≤0.5", I: "1^", R: "≥2" },
                        comments: {
                            exam: ["The most sensitive carbapenem for screening for carbapenemase production in Enterobacterales.", "Has a narrower spectrum than other carbapenems; lacks reliable activity against *Pseudomonas* and *Acinetobacter*.", "Resistance to ertapenem alone may indicate an ESBL/AmpC with porin loss, whereas resistance to multiple carbapenems strongly suggests a carbapenemase."],
                            mechanism: ["Group 1 carbapenem. Resistance mechanisms are the same as for other carbapenems."]
                        }
                    },
                    {
                        antimicrobial: "Imipenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥23", I: "20-22^", R: "≤19" },
                        mic: { S: "≤1", I: "2^", R: "≥4" },
                        comments: {
                            exam: ["A broad-spectrum carbapenem.", "*Proteus*, *Providencia*, and *Morganella* spp. may have intrinsically higher MICs to imipenem that do not reflect carbapenemase production."],
                            mechanism: ["Group 2 carbapenem. Resistance mechanisms are the same as for other carbapenems."]
                        }
                    },
                    {
                        antimicrobial: "Meropenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥23", I: "20-22^", R: "≤19" },
                        mic: { S: "≤1", I: "2^", R: "≥4" },
                        comments: {
                            exam: ["A broad-spectrum carbapenem, often a workhorse for severe nosocomial infections.", "Isolates with intermediate or resistant results warrant consultation with an ID specialist. High-dose, extended-infusion regimens may be considered.", "Presence of a carbapenemase (e.g., KPC, NDM, OXA-48) is a critical finding for infection control and therapeutic choice."],
                            mechanism: ["Group 2 carbapenem. Resistance is primarily mediated by carbapenemase production or a combination of porin loss and other β-lactamases."]
                        }
                    },
                    {
                        antimicrobial: "Amikacin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥20", I: "17-19^", R: "≤16" },
                        mic: { S: "≤4", I: "8^", R: "≥16" },
                        comments: {
                            exam: ["Broadest spectrum aminoglycoside, often active against isolates resistant to gentamicin and tobramycin.", "Therapeutic drug monitoring is common to ensure efficacy (Peak/MIC ratio) and minimize nephrotoxicity/ototoxicity."],
                            mechanism: ["Inhibits protein synthesis by binding to 30S ribosome. Resistance via aminoglycoside-modifying enzymes (AMEs), 16S rRNA methyltransferases (conferring high-level resistance), or efflux pumps."]
                        }
                    },
                    {
                        antimicrobial: "Ciprofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥26", I: "22-25^", R: "≤21" },
                        mic: { S: "≤0.25", I: "0.5^", R: "≥1" },
                        comments: {
                            exam: ["High rates of resistance worldwide limit its empiric use for serious infections.", "Breakpoints for *Salmonella* spp. are different and should be consulted separately. This breakpoint does not apply to them.", "Useful for UTIs due to excellent urinary concentration, but resistance is still a major issue."],
                            mechanism: ["A fluoroquinolone that inhibits bacterial DNA synthesis by targeting DNA gyrase (topoisomerase II) and topoisomerase IV.", "Resistance occurs through two main mechanisms: 1) Point mutations in the quinolone-resistance determining region (QRDR) of the target enzyme genes (*gyrA*, *parC*). 2) Plasmid-mediated resistance genes (*qnr*) or efflux pumps."]
                        }
                    },
                    {
                        antimicrobial: "Trimethoprim-sulfamethoxazole",
                        diskContent: "1.25/23.75 µg",
                        zoneDiameter: { S: "≥16", R: "≤10" },
                        mic: { S: "≤2/38", R: "≥4/76" },
                        comments: {
                            exam: ["Key agent for UTIs and for treating *Stenotrophomonas maltophilia* and *Nocardia*.", "Activity against Enterobacterales is variable and often low in hospital settings.", "Testing media must be low in thymidine to avoid false resistance."],
                            mechanism: ["Sequential blockade of the folate synthesis pathway. Resistance via acquisition of resistant target enzymes (*dfr*, *sul* genes)."]
                        }
                    },
                    {
                        antimicrobial: "Colistin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A", I: "N/A", R: "N/A" },
                        mic: { S: "N/A", I: "≤2", R: "≥4" },
                        comments: {
                            exam: ["A last-resort polypeptide antibiotic for multidrug-resistant Gram-negative infections.", "<u>**Disk diffusion and gradient diffusion are unreliable and must not be used.**</u> MIC testing must be performed by reference broth microdilution.", "Associated with significant nephrotoxicity and neurotoxicity. Therapeutic drug monitoring is advised.", "**Intrinsic resistance** is seen in *Proteus*, *Providencia*, *Morganella*, and *Serratia* spp."],
                            mechanism: ["Acts as a cationic detergent, binding to the lipid A portion of lipopolysaccharide (LPS) in the outer membrane, displacing Ca2+ and Mg2+ ions and disrupting membrane integrity.", "Acquired resistance is most commonly due to modifications of the lipid A moiety (e.g., addition of phosphoethanolamine), mediated by chromosomal mutations or plasmid-borne *mcr* genes."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Salmonella and Shigella spp.",
                aliases: ["salmonella", "shigella", "s. typhi", "s. sonnei"],
                data: [
                    {
                        antimicrobial: "Ampicillin",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥17", I: "14-16^", R: "≤13" },
                        mic: { S: "≤8", I: "16^", R: "≥32" },
                        comments: {
                            exam: ["A primary agent for susceptible isolates.", "Resistance is common.", "Predicts amoxicillin susceptibility."],
                            mechanism: ["Resistance primarily via β-lactamase production (e.g., TEM-1)."]
                        }
                    },
                    {
                        antimicrobial: "Ciprofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥31", I: "21-30^", R: "≤20" },
                        mic: { S: "≤0.06", I: "0.12-0.5^", R: "≥1" },
                        comments: {
                            exam: ["Drug of choice for typhoid fever and severe salmonellosis/shigellosis.", "<u>**Decreased ciprofloxacin susceptibility (DCS)**</u>, even with MICs in the 'susceptible' range (e.g., 0.12-0.5 µg/mL), is associated with clinical failure. Pefloxacin disk screening can help detect DCS.", "Any resistance is clinically significant."],
                            mechanism: ["Resistance via mutations in *gyrA* is the most significant mechanism."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥23", I: "20-22^", R: "≤19" },
                        mic: { S: "≤1", I: "2^", R: "≥4" },
                        comments: {
                            exam: ["Alternative drug of choice, especially for fluoroquinolone-resistant strains or in children.", "Resistance (usually via ESBLs like CTX-M) is an increasing global problem."],
                            mechanism: ["Resistance via ESBL production."]
                        }
                    },
                    {
                        antimicrobial: "Azithromycin",
                        diskContent: "15 µg",
                        zoneDiameter: { S: "≥13", R: "≤12" },
                        mic: { S: "≤16", R: "≥32" },
                        comments: {
                            exam: ["An important oral option, particularly for uncomplicated typhoid fever and resistant shigellosis.", "Breakpoints apply only to *S. enterica* ser. Typhi and *Shigella* spp.", "Disk diffusion can be difficult to read; MIC testing is preferred if zones are unclear."],
                            mechanism: ["Resistance via efflux pumps (*mphA*) or target site modification (*erm*)."]
                        }
                    },
                    {
                        antimicrobial: "Trimethoprim-sulfamethoxazole",
                        diskContent: "1.25/23.75 µg",
                        zoneDiameter: { S: "≥16", R: "≤10" },
                        mic: { S: "≤2/38", R: "≥4/76" },
                        comments: {
                            exam: ["High rates of resistance globally have limited its use.", "Still an option if known to be susceptible."],
                            mechanism: ["Resistance via acquisition of *dfr* and *sul* genes."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Pseudomonas aeruginosa",
                aliases: ["pseudomonas", "p. aeruginosa"],
                data: [
                    {
                        antimicrobial: "Piperacillin-tazobactam",
                        diskContent: "100/10 µg",
                        zoneDiameter: { S: "≥22", I: "18-21", R: "≤17" },
                        mic: { S: "≤16/4", I: "32/4", R: "≥64/4" },
                        comments: {
                           exam: ["A workhorse anti-pseudomonal agent. Extended infusions (over 3-4 hours) are standard practice to optimize pharmacodynamics (%T > MIC).", "Tazobactam does not inhibit the chromosomal AmpC β-lactamase of *P. aeruginosa*, which can be derepressed.", "The intermediate category is a buffer zone; clinical utility for these isolates is uncertain."],
                           mechanism: ["Resistance is common and occurs via: 1) High-level, stable derepression of the chromosomal AmpC β-lactamase. 2) Upregulation of efflux pumps (e.g., MexAB-OprM). 3) Acquired β-lactamases (ESBLs, carbapenemases)."]
                        }
                    },
                    {
                        antimicrobial: "Ceftazidime",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥18", I: "15-17^", R: "≤14" },
                        mic: { S: "≤8", I: "16^", R: "≥32" },
                        comments: {
                           exam: ["A third-generation cephalosporin with reliable anti-pseudomonal activity, though resistance is common.", "Often used in combination with an aminoglycoside for serious infections.", "Susceptibility to ceftazidime does not predict susceptibility to other cephalosporins."],
                           mechanism: ["Resistance mechanisms are similar to piperacillin-tazobactam: AmpC derepression, efflux pumps, and acquired β-lactamases (ESBLs, carbapenemases)."]
                        }
                    },
                    {
                        antimicrobial: "Cefepime",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥18", I: "15-17^", R: "≤14" },
                        mic: { S: "≤8", I: "16^", R: "≥32" },
                        comments: {
                           exam: ["A 4th-generation cephalosporin that is more stable to hydrolysis by AmpC than ceftazidime.", "Often used for serious nosocomial infections. Extended infusions can optimize therapy."],
                           mechanism: ["Resistance via efflux pumps, high-level AmpC, and acquired β-lactamases (ESBLs, carbapenemases)."]
                        }
                    },
                    {
                        antimicrobial: "Imipenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥19", I: "16-18^", R: "≤15" },
                        mic: { S: "≤2", I: "4^", R: "≥8" },
                        comments: {
                           exam: ["First carbapenem with excellent activity, but resistance is now common.", "Resistance often mediated by loss of the OprD porin, which makes the isolate resistant to imipenem but potentially still susceptible to meropenem."],
                           mechanism: ["Resistance via OprD loss, efflux pumps, and carbapenemases (especially MBLs)."]
                        }
                    },
                    {
                        antimicrobial: "Meropenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥19", I: "16-18^", R: "≤15" },
                        mic: { S: "≤2", I: "4^", R: "≥8" },
                        comments: {
                           exam: ["Resistance can develop *during* therapy, a hallmark of *P. aeruginosa*. Repeat susceptibility testing may be warranted in non-responding patients.", "Often used as a cornerstone of anti-pseudomonal therapy, especially in critically ill patients. Combination therapy is frequently recommended to prevent resistance.", "Detection of metallo-β-lactamases (MBLs) like VIM or IMP is critical, as it confers resistance to all carbapenems and most other β-lactams."],
                           mechanism: ["Mechanisms of resistance are multifactorial and often cumulative:", "1. **Reduced uptake:** Downregulation of the OprD porin is a common cause of resistance.", "2. **Efflux pumps:** Upregulation of pumps like MexAB-OprM can extrude the drug.", "3. **Enzymatic degradation:** Production of carbapenemases (MBLs, KPC, etc.).", "4. **PBP alterations:** Less common but can contribute."]
                        }
                    },
                    {
                        antimicrobial: "Amikacin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥17", I: "15-16^", R: "≤14" },
                        mic: { S: "≤16", I: "32^", R: "≥64" },
                        comments: {
                           exam: ["An aminoglycoside often reserved for MDR *P. aeruginosa* as it is less susceptible to many common aminoglycoside-modifying enzymes (AMEs).", "High-dose, once-daily (or 'extended interval') dosing is used to maximize the Peak/MIC ratio, which predicts efficacy, and to minimize toxicity.", "Breakpoints listed are for urinary tract isolates only, reflecting high achievable concentrations in urine."],
                           mechanism: ["Binds to the 30S ribosomal subunit, causing mistranslation of mRNA and inhibiting protein synthesis.", "Resistance is primarily due to AMEs, efflux pumps (MexXY-OprM), and, less commonly, mutations in the 16S rRNA target site."]
                        }
                    },
                    {
                        antimicrobial: "Ciprofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥25", I: "19-24^", R: "≤18" },
                        mic: { S: "≤0.5", I: "1^", R: "≥2" },
                        comments: {
                           exam: ["Resistance is very common. Should not be used empirically for serious infections without local antibiogram support.", "Excellent oral bioavailability is an advantage for step-down therapy."],
                           mechanism: ["Mutations in DNA gyrase (*gyrA*) and topoisomerase IV (*parC*) are the primary mechanisms. Efflux pumps also contribute significantly."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Acinetobacter baumannii complex",
                aliases: ["acinetobacter", "a. baumannii"],
                data: [
                    {
                        antimicrobial: "Meropenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥18", I: "15-17", R: "≤14" },
                        mic: { S: "≤2", I: "4", R: "≥8" },
                        comments: {
                            exam: ["Carbapenem resistance is extremely common and a defining feature of problematic *A. baumannii* infections.", "Resistance is most often due to production of OXA-type carbapenemases (e.g., OXA-23, OXA-24/40, OXA-58).", "Therapeutic options for carbapenem-resistant *A. baumannii* (CRAB) are very limited, often requiring agents like colistin or tigecycline."],
                            mechanism: ["Primary mechanism is production of carbapenem-hydrolyzing class D β-lactamases (CHDLs), i.e., OXA enzymes. Other mechanisms like MBLs, porin loss, and efflux pumps also contribute."]
                        }
                    },
                    {
                        antimicrobial: "Ampicillin-sulbactam",
                        diskContent: "10/10 µg",
                        zoneDiameter: { S: "≥22", I: "17-21", R: "≤16" },
                        mic: { S: "≤8/4", I: "16/8", R: "≥32/16" },
                        comments: {
                            exam: ["Sulbactam itself has intrinsic activity against *A. baumannii* by binding to its PBPs. The ampicillin component is largely irrelevant.", "High-dose ampicillin-sulbactam is often a key component of therapy for susceptible isolates.", "Resistance is common."],
                            mechanism: ["Activity is from sulbactam's affinity for *Acinetobacter* PBPs. Resistance occurs via production of β-lactamases (like TEM-1 which sulbactam can inhibit, or OXA carbapenemases which it cannot) or alterations in PBPs."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Stenotrophomonas maltophilia",
                aliases: ["stenotrophomonas", "s. maltophilia"],
                data: [
                    {
                        antimicrobial: "Trimethoprim-sulfamethoxazole",
                        diskContent: "1.25/23.75 µg",
                        zoneDiameter: { S: "≥16", R: "≤10" },
                        mic: { S: "≤2/38", R: "≥4/76" },
                        comments: {
                            exam: ["<u>**This is the drug of choice for *S. maltophilia* infections.**</u>", "Resistance, while emerging, is still relatively uncommon compared to other agents.", "High-dose therapy is typically recommended for serious infections."],
                            mechanism: ["Inhibits folate synthesis. Resistance is mediated by *sul* genes (especially *sul1*) and *dfrA* genes, often carried on integrons."]
                        }
                    },
                    {
                        antimicrobial: "Levofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥17", I: "14-16", R: "≤13" },
                        mic: { S: "≤2", I: "4", R: "≥8" },
                        comments: {
                            exam: ["An alternative therapeutic option, though resistance rates are higher than for TMP-SMX.", "Combination therapy is often considered for severe infections."],
                            mechanism: ["Resistance via mutations in DNA gyrase and topoisomerase IV, and through the SmeDEF efflux pump."]
                        }
                    },
                    {
                        antimicrobial: "Minocycline",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥26", I: "21-25", R: "≤20" },
                        mic: { S: "≤1", I: "2", R: "≥4" },
                        comments: {
                            exam: ["Retains good in vitro activity and is another important alternative agent.", "It is a tetracycline, but intrinsic resistance to tetracycline itself does not apply to minocycline for this organism."],
                            mechanism: ["Inhibits protein synthesis by binding to the 30S ribosome. Resistance is primarily via efflux pumps."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Burkholderia cepacia complex",
                aliases: ["burkholderia", "b. cepacia"],
                data: [
                    {
                        antimicrobial: "Susceptibility Testing Note",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "N/A" },
                        comments: {
                           exam: ["<u>**CLSI has removed and archived clinical breakpoints for *B. cepacia* complex.**</u> This is a critical point.", "AST methods (broth microdilution, agar dilution, disk diffusion) show poor correlation and are considered unreliable for predicting clinical outcomes.", "Reporting should state that AST is not routinely performed due to lack of accurate methods. Treatment decisions should be guided by expert consultation, not in vitro results."],
                           mechanism: ["These organisms possess multiple, complex, and often inducible resistance mechanisms, making in vitro testing highly problematic and poorly predictive of in vivo activity."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Staphylococcus spp.",
                aliases: ["staphylococcus", "staph", "s. aureus", "staphylococcus aureus", "mrsa", "mssa"],
                data: [
                    {
                        antimicrobial: "Cefoxitin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥22", R: "≤21" },
                        mic: { S: "≤4", R: "≥8" },
                        comments: {
                           exam: ["<u>**This is a surrogate test, NOT a therapeutic option for *S. aureus*.**</u>", "The cefoxitin disk test is the recommended phenotypic method for detecting *mecA*-mediated oxacillin resistance (MRSA).", "A cefoxitin-resistant result should be reported as 'Methicillin (Oxacillin) Resistant'."],
                           mechanism: ["Cefoxitin is a better inducer of the *mecA* gene expression in vitro compared to oxacillin, making the resistance phenotype easier to detect.", "Resistance to cefoxitin in this context is a proxy for the presence of PBP2a, the altered penicillin-binding protein encoded by *mecA*."]
                        }
                    },
                    {
                        antimicrobial: "Vancomycin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "N/A", I: "N/A", R: "N/A" },
                        mic: { S: "≤2", I: "4-8", R: "≥16" },
                        comments: {
                            exam: ["<u>**MIC testing is mandatory;** disk diffusion is unreliable.</u>", "For serious infections like bacteremia, an MIC of 2 µg/mL is associated with higher rates of clinical failure ('MIC creep'). Alternative therapies should be considered.", "Therapeutic drug monitoring is crucial. The target is an **AUC/MIC ratio of 400-600** for clinical efficacy while minimizing nephrotoxicity.", "VISA (Vancomycin-Intermediate *S. aureus*, MIC 4-8) and hVISA (hetero-VISA) are significant clinical challenges. VRSA (Vancomycin-Resistant, MIC ≥16) is rare but a major public health concern."],
                            mechanism: ["Vancomycin inhibits cell wall synthesis by binding to the D-Ala-D-Ala termini of peptidoglycan precursors.", "Resistance in VISA/hVISA is typically due to a **thickened, disorganized cell wall** that traps vancomycin molecules before they reach their target.", "High-level resistance (VRSA) is due to acquisition of the *vanA* gene cluster (usually from *Enterococcus*) which alters the peptidoglycan target to D-Ala-D-Lac, to which vancomycin cannot bind."]
                        }
                    },
                    {
                        antimicrobial: "Daptomycin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤1" },
                        comments: {
                           exam: ["<u>**Do not use for pneumonia.**</u> Daptomycin is inactivated by pulmonary surfactant.", "Testing requires **calcium supplementation** in the media (50 µg/mL). Failure to do so can lead to falsely elevated MICs.", "Excellent choice for MRSA bacteremia and endocarditis, especially if vancomycin MIC is high (≥2 µg/mL).", "Resistance can emerge on therapy, often associated with mutations in genes like *mprF*."],
                           mechanism: ["Daptomycin is a cyclic lipopeptide that inserts into the bacterial cell membrane in a calcium-dependent manner.", "This insertion leads to membrane depolarization, potassium efflux, and rapid bactericidal activity without cell lysis."]
                        }
                    },
                    {
                        antimicrobial: "Linezolid",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥26", I: "23-25", R: "≤22" },
                        mic: { S: "≤4", R: "≥8" },
                        comments: {
                           exam: ["An oxazolidinone with excellent activity against most Gram-positives, including MRSA and VRE.", "Primarily bacteriostatic against staphylococci. A good option for pneumonia (including VAP/HAP) and skin/soft tissue infections.", "<u>**Major side effect with prolonged use (>2 weeks) is myelosuppression (especially thrombocytopenia).**</u>", "Resistance is rare but increasing. Any resistant isolate is significant and warrants infection control attention."],
                           mechanism: ["Unique mechanism: binds to the P site of the 50S ribosomal subunit, preventing the formation of the 70S initiation complex. This prevents protein synthesis at a very early stage.", "Resistance is typically due to point mutations in the 23S rRNA gene. Less commonly, the plasmid-mediated *cfr* gene can confer resistance to linezolid, phenicols, lincosamides, pleuromutilins, and streptogramin A."]
                        }
                    },
                    {
                        antimicrobial: "Clindamycin",
                        diskContent: "2 µg",
                        zoneDiameter: { S: "≥21", I: "15-20", R: "≤14" },
                        mic: { S: "≤0.5", I: "1-2", R: "≥4" },
                        comments: {
                           exam: ["Excellent tissue penetration, making it useful for skin/soft tissue infections and anaerobic infections.", "<u>**The D-test is mandatory**</u> for erythromycin-resistant, clindamycin-susceptible isolates to detect inducible resistance mediated by the *erm* gene. A positive D-test means clindamycin should be reported as resistant.", "Associated with a high risk of *Clostridioides difficile* infection."],
                           mechanism: ["A lincosamide that binds to the 50S ribosomal subunit, inhibiting protein synthesis. Can also suppress toxin production.", "Resistance via *erm* genes (target site modification), *msrA* gene (efflux), or enzymatic inactivation."]
                        }
                    },
                    {
                        antimicrobial: "Trimethoprim-sulfamethoxazole",
                        diskContent: "1.25/23.75 µg",
                        zoneDiameter: { S: "≥16", R: "≤10" },
                        mic: { S: "≤2/38", R: "≥4/76" },
                        comments: {
                           exam: ["Drug of choice for uncomplicated skin and soft tissue infections caused by **community-acquired MRSA (CA-MRSA)**, which is typically susceptible.", "Not recommended for severe, invasive MRSA infections (e.g., bacteremia) due to potential for in vivo resistance and limited bactericidal activity.", "Thymidine in testing media can interfere with results, leading to false resistance. Use low-thymidine media (e.g., Mueller-Hinton agar)."],
                           mechanism: ["Acts by sequential blockade of the folic acid synthesis pathway. Sulfamethoxazole inhibits dihydropteroate synthase, and trimethoprim inhibits dihydrofolate reductase.", "Resistance is common and mediated by acquisition of alternative, resistant forms of the target enzymes, encoded by *dfr* (trimethoprim) and *sul* (sulfamethoxazole) genes."]
                        }
                    },
                    {
                        antimicrobial: "Rifampin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥20", I: "17-19", R: "≤16" },
                        mic: { S: "≤1", I: "2", R: "≥4" },
                        comments: {
                           exam: ["<u>**Must NEVER be used as monotherapy**</u> for staphylococcal infections due to the rapid emergence of resistance.", "Used as part of a combination regimen, particularly for prosthetic device infections due to its excellent biofilm penetration."],
                           mechanism: ["Inhibits bacterial DNA-dependent RNA polymerase. Resistance develops rapidly via a single point mutation in the *rpoB* gene."]
                        }
                    },
                    {
                        antimicrobial: "Dalbavancin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤0.25" },
                        comments: {
                           exam: ["A long-acting lipoglycopeptide with an extremely long half-life, allowing for once-weekly or even single-dose administration.", "Approved for acute bacterial skin and skin structure infections (ABSSSI).", "Breakpoints apply to *S. aureus* only."],
                           mechanism: ["Inhibits cell wall synthesis by binding to D-Ala-D-Ala precursors (like vancomycin) and anchoring to the cell membrane via its lipid tail."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Enterococcus faecalis",
                aliases: ["e. faecalis"],
                data: [
                    {
                        antimicrobial: "Fosfomycin",
                        diskContent: "200 µg",
                        zoneDiameter: { S: "≥16", I: "13-15", R: "≤12" },
                        mic: { S: "≤64", I: "128", R: "≥256" },
                        comments: {
                           exam: ["<u>**Breakpoints are for uncomplicated UTIs ONLY.**</u>", "Activity against *E. faecium* is poor; these breakpoints do not apply.", "MIC testing requires agar dilution with media supplemented with glucose-6-phosphate."],
                           mechanism: ["Inhibits an early step in cell wall synthesis by inactivating the MurA enzyme. Resistance via mutations in the target enzyme or impaired drug transport."]
                        }
                    },
                    {
                        antimicrobial: "Dalbavancin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤0.25" },
                        comments: {
                           exam: ["Breakpoints apply only to **vancomycin-susceptible** *E. faecalis*."],
                           mechanism: ["Inhibits cell wall synthesis."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Enterococcus faecium",
                aliases: ["e. faecium"],
                data: [
                    {
                        antimicrobial: "Daptomycin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { SDD: "≤4", R: "≥8" },
                        comments: {
                           exam: ["Key bactericidal agent for VRE infections caused by *E. faecium*.", "Note there is no 'Susceptible' category, only 'Susceptible-Dose Dependent' (SDD), which requires high doses (8-12 mg/kg/day) for efficacy.", "Resistance can emerge on therapy."],
                           mechanism: ["Calcium-dependent membrane disruption. Resistance mechanisms are complex and involve changes to cell membrane charge and fluidity."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Enterococcus spp.",
                aliases: ["enterococcus", "vre"],
                data: [
                    {
                        antimicrobial: "Ampicillin",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥17", R: "≤16" },
                        mic: { S: "≤8", R: "≥16" },
                        comments: {
                            exam: ["Drug of choice for susceptible *E. faecalis* infections.", "*E. faecium* is frequently resistant.", "Ampicillin susceptibility predicts susceptibility to amoxicillin and, for non-β-lactamase producers, ampicillin-sulbactam and piperacillin-tazobactam."],
                            mechanism: ["Resistance is typically due to altered PBPs with low affinity for ampicillin. β-lactamase production is very rare."]
                        }
                    },
                    {
                        antimicrobial: "Vancomycin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥17", I: "15-16", R: "≤14" },
                        mic: { S: "≤4", I: "8-16", R: "≥32" },
                        comments: {
                            exam: ["**VRE (Vancomycin-Resistant Enterococci)** is a major nosocomial pathogen. *E. faecium* is more commonly resistant than *E. faecalis*.", "<u>**WARNING:**</u> Cephalosporins, clindamycin, and TMP-SMX are **clinically ineffective** against enterococci, regardless of in vitro results.", "Intrinsic low-level resistance (*vanC*) in *E. gallinarum* and *E. casseliflavus* must be differentiated from acquired high-level resistance (*vanA*, *vanB*) which has greater clinical and infection control implications."],
                            mechanism: ["Acquired resistance is mediated by gene clusters (*vanA*, *vanB*, etc.) that alter the peptidoglycan synthesis pathway.", "**VanA:** High-level, inducible resistance to both vancomycin and teicoplanin.", "**VanB:** Variable-level, inducible resistance to vancomycin only (remains susceptible to teicoplanin)."]
                        }
                    },
                    {
                        antimicrobial: "Linezolid",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥23", I: "21-22", R: "≤20" },
                        mic: { S: "≤2", I: "4", R: "≥8" },
                        comments: {
                           exam: ["A key therapeutic option for VRE infections, especially those caused by *E. faecium*.", "Generally bacteriostatic against enterococci.", "Resistance is emerging and is a serious concern. Any linezolid-resistant enterococcus should be reported to infection control."],
                           mechanism: ["Inhibits protein synthesis by binding to the 50S ribosomal subunit. Resistance is primarily via mutations in the 23S rRNA gene or acquisition of the *cfr* or *optrA* genes."]
                        }
                    },
                    {
                        antimicrobial: "Gentamicin (High-Level)",
                        diskContent: "120 µg",
                        zoneDiameter: { S: "≥10", I: "7-9", R: "≤6" },
                        mic: { S: "≤500", R: ">500" },
                        comments: {
                           exam: ["This is a **synergy screen**, not a test for monotherapy susceptibility.", "Susceptibility to high-level gentamicin predicts synergy with a cell-wall active agent (like ampicillin or vancomycin) for treating serious infections like endocarditis.", "If resistant, synergy is lost."],
                           mechanism: ["High-level resistance (HLR) is mediated by aminoglycoside-modifying enzymes that can inactivate the high drug concentrations achieved inside the bacterium with synergistic therapy."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Streptococcus pneumoniae",
                aliases: ["strep pneumo", "s. pneumoniae", "pneumococcus"],
                data: [
                    {
                        antimicrobial: "Penicillin",
                        diskContent: "1 µg Oxacillin (screen)",
                        zoneDiameter: { S: "≥20 (Oxacillin screen)" },
                        mic: { S: "≤0.06 (Meningitis) / ≤2 (Non-meningitis)", I: "0.12-1 (Oral) / 4 (Non-meningitis)", R: "≥0.12 (Meningitis) / ≥2 (Oral) / ≥8 (Non-meningitis)" },
                        comments: {
                           exam: ["Breakpoints are route and site-dependent (Oral vs. Parenteral Non-Meningitis vs. Meningitis). This is a classic exam topic.", "An oxacillin disk screen is used: a zone ≥20mm predicts penicillin susceptibility (MIC ≤0.06 µg/mL). If zone is ≤19mm, a penicillin MIC must be performed.", "Penicillin resistance is now less common due to the widespread use of the pneumococcal conjugate vaccine (PCV)."],
                           mechanism: ["Resistance is not due to β-lactamase production. It is mediated by alterations in the structure of Penicillin-Binding Proteins (PBPs), which reduces their affinity for penicillin."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥30", R: "N/A" },
                        mic: { S: "≤0.5 (Meningitis) / ≤1 (Non-meningitis)", I: "1 (Meningitis) / 2 (Non-meningitis)", R: "≥2 (Meningitis) / ≥4 (Non-meningitis)" },
                        comments: {
                           exam: ["Like penicillin, breakpoints are site-dependent (Meningitis vs. Non-Meningitis).", "Empiric therapy of choice for community-acquired pneumonia and bacterial meningitis.", "Ceftriaxone resistance often correlates with high-level penicillin resistance."],
                           mechanism: ["Resistance is mediated by the same PBP alterations that cause penicillin resistance."]
                        }
                    },
                    {
                        antimicrobial: "Levofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥17", I: "14-16", R: "≤13" },
                        mic: { S: "≤2", I: "4", R: "≥8" },
                        comments: {
                           exam: ["A 'respiratory fluoroquinolone' with good activity against *S. pneumoniae*.", "Resistance is a concern, especially in patients with prior fluoroquinolone exposure.", "Susceptibility to levofloxacin predicts susceptibility to moxifloxacin."],
                           mechanism: ["Resistance develops via stepwise mutations in DNA gyrase (*gyrA*) and topoisomerase IV (*parC*)."]
                        }
                    },
                    {
                        antimicrobial: "Vancomycin",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥17" },
                        mic: { S: "≤1" },
                        comments: {
                           exam: ["<u>**Vancomycin resistance in *S. pneumoniae* has not been reliably documented.**</u>", "Any isolate appearing resistant should be re-identified and re-tested. If confirmed, it is a critical result requiring public health notification."],
                           mechanism: ["N/A"]
                        }
                    }
                ]
            },
            {
                organismGroup: "Streptococcus spp. (β-Hemolytic Group)",
                aliases: ["beta-hemolytic strep", "strep pyogenes", "strep agalactiae", "group a strep", "group b strep"],
                data: [
                    {
                        antimicrobial: "Penicillin",
                        diskContent: "10 units",
                        zoneDiameter: { S: "≥24" },
                        mic: { S: "≤0.12" },
                        comments: {
                           exam: ["<u>**Penicillin resistance has never been documented in *S. pyogenes* (Group A).**</u> Routine testing is not necessary.", "All β-hemolytic streptococci are considered universally susceptible to penicillin and ampicillin. Any non-susceptible result is highly unusual and requires confirmation.", "Penicillin remains the drug of choice for pharyngitis and other GAS infections."],
                           mechanism: ["N/A - No clinically significant resistance mechanism has emerged."]
                        }
                    },
                    {
                        antimicrobial: "Erythromycin",
                        diskContent: "15 µg",
                        zoneDiameter: { S: "≥21", I: "16-20", R: "≤15" },
                        mic: { S: "≤0.25", I: "0.5", R: "≥1" },
                        comments: {
                           exam: ["Macrolide resistance is common and testing is necessary if it is being considered for therapy (e.g., in penicillin-allergic patients).", "Erythromycin resistance predicts resistance to azithromycin and clarithromycin.", "Resistance is a key factor in deciding whether to use clindamycin, as it necessitates the D-test."],
                           mechanism: ["Resistance via target site modification (*erm* genes) or efflux (*mef* gene)."]
                        }
                    },
                    {
                        antimicrobial: "Clindamycin",
                        diskContent: "2 µg",
                        zoneDiameter: { S: "≥19", I: "16-18", R: "≤15" },
                        mic: { S: "≤0.25", I: "0.5", R: "≥1" },
                        comments: {
                           exam: ["Used for its anti-toxin effect in severe infections like necrotizing fasciitis and streptococcal toxic shock syndrome.", "<u>**D-test is mandatory**</u> for erythromycin-resistant isolates to check for inducible resistance."],
                           mechanism: ["Resistance via *erm* genes (inducible or constitutive) or target site mutations."]
                        }
                    },
                    {
                        antimicrobial: "Dalbavancin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤0.25" },
                        comments: {
                           exam: ["A long-acting lipoglycopeptide. Breakpoints apply to *S. pyogenes*, *S. agalactiae*, and *S. dysgalactiae*."],
                           mechanism: ["Inhibits cell wall synthesis."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Streptococcus spp. (Viridans Group)",
                aliases: ["viridans strep", "strep mitis", "strep anginosus"],
                data: [
                     {
                        antimicrobial: "Penicillin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤0.12", I: "0.25-2", R: "≥4" },
                        comments: {
                           exam: ["<u>**MIC testing is required; disk diffusion is not reliable.**</u>", "Unlike β-hemolytic strep, penicillin resistance is common in viridans group streptococci, especially in the *S. mitis* group.", "Breakpoints are crucial for guiding therapy for endocarditis, a common infection caused by these organisms."],
                           mechanism: ["Resistance is mediated by alterations in Penicillin-Binding Proteins (PBPs), similar to *S. pneumoniae*."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥27", I: "25-26", R: "≤24" },
                        mic: { S: "≤1", I: "2", R: "≥4" },
                        comments: {
                           exam: ["A key agent for treating endocarditis caused by penicillin-resistant viridans streptococci.", "Often used in combination with gentamicin for synergistic bactericidal activity in endocarditis treatment."],
                           mechanism: ["Resistance via altered PBPs."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Haemophilus influenzae",
                aliases: ["h. influenzae", "haemophilus"],
                data: [
                    {
                        antimicrobial: "Ampicillin",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥22", I: "19-21", R: "≤18" },
                        mic: { S: "≤1", I: "2", R: "≥4" },
                        comments: {
                           exam: ["Resistance is common. A β-lactamase test is a rapid way to detect the most common resistance mechanism.", "BLNAR (β-lactamase-negative, ampicillin-resistant) strains exist due to PBP modifications and are resistant to amoxicillin-clavulanate."],
                           mechanism: ["Resistance is primarily due to production of TEM-1 or ROB-1 β-lactamases. BLNAR strains have altered PBPs."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥26" },
                        mic: { S: "≤2" },
                        comments: {
                           exam: ["Drug of choice for serious infections like meningitis and epiglottitis caused by *H. influenzae*.", "Resistance is extremely rare. Any non-susceptible isolate should be confirmed."],
                           mechanism: ["N/A - Clinically significant resistance is not observed."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Neisseria gonorrhoeae",
                aliases: ["gonococcus", "n. gonorrhoeae", "gc"],
                data: [
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥35" },
                        mic: { S: "≤0.25" },
                        comments: {
                           exam: ["The cornerstone of current gonorrhea treatment regimens, typically given as a single IM injection.", "Susceptibility surveillance is critical due to the organism's ability to acquire resistance. Decreased susceptibility (elevated MICs) is a major public health concern."],
                           mechanism: ["Resistance is emerging and is due to a mosaic *penA* gene encoding an altered PBP2, which reduces affinity for ceftriaxone."]
                        }
                    },
                    {
                        antimicrobial: "Azithromycin",
                        diskContent: "15 µg",
                        zoneDiameter: { S: "≥30" },
                        mic: { S: "≤1" },
                        comments: {
                           exam: ["Previously used in combination with ceftriaxone, but this is no longer routinely recommended in many regions due to rising resistance.", "High-level resistance is a significant concern."],
                           mechanism: ["Resistance is due to mutations in the 23S rRNA gene or acquisition of efflux pumps."]
                        }
                    },
                    {
                        antimicrobial: "Ciprofloxacin",
                        diskContent: "5 µg",
                        zoneDiameter: { S: "≥41", I: "28-40", R: "≤27" },
                        mic: { S: "≤0.06", I: "0.12-0.5", R: "≥1" },
                        comments: {
                           exam: ["<u>**Fluoroquinolones are no longer recommended for empiric treatment of gonorrhea due to widespread high-level resistance.**</u>", "Testing is performed for surveillance purposes."],
                           mechanism: ["High-level resistance is due to mutations in the *gyrA* and *parC* genes."]
                        }
                    }
                ]
            },
            {
                organismGroup: "Neisseria meningitidis",
                aliases: ["meningococcus", "n. meningitidis"],
                data: [
                    {
                        antimicrobial: "Penicillin",
                        diskContent: "N/A",
                        zoneDiameter: { S: "N/A" },
                        mic: { S: "≤0.06", I: "0.12-0.25", R: "≥0.5" },
                        comments: {
                           exam: ["Penicillin resistance, while still uncommon, is increasing. Susceptibility testing is essential for invasive isolates.", "Resistance is chromosomally mediated, not via β-lactamase in most cases."],
                           mechanism: ["Resistance is due to alterations in the *penA* gene, leading to a modified PBP2 with reduced affinity for penicillin."]
                        }
                    },
                    {
                        antimicrobial: "Ceftriaxone",
                        diskContent: "30 µg",
                        zoneDiameter: { S: "≥34" },
                        mic: { S: "≤0.12" },
                        comments: {
                           exam: ["Drug of choice for treating meningococcal disease.", "Resistance is exceptionally rare. Any non-susceptible isolate is a critical finding and must be confirmed and reported to public health authorities."],
                           mechanism: ["N/A - Clinically significant resistance is not observed."]
                        }
                    },
                    {
                        antimicrobial: "Meropenem",
                        diskContent: "10 µg",
                        zoneDiameter: { S: "≥30" },
                        mic: { S: "≤0.25" },
                        comments: {
                           exam: ["An alternative for treatment, especially if there is concern for resistance to third-generation cephalosporins.", "Resistance is very rare."],
                           mechanism: ["N/A"]
                        }
                    }
                ]
            }
        ];
        
        const intrinsicResistanceData = [
            {
                organism: "Citrobacter freundii complex",
                aliases: ["citrobacter freundii", "c. freundii"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Ampicillin-sulbactam", "Ticarcillin", "Cephalosporins (1st Gen, e.g., Cefazolin)", "Cefuroxime (2nd Gen)"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Klebsiella pneumoniae",
                aliases: ["klebsiella", "k. pneumoniae"],
                resistances: ["Ampicillin"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Enterobacter cloacae complex",
                aliases: ["enterobacter", "e. cloacae"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Ampicillin-sulbactam", "Cephalosporins (1st & 2nd Gen, e.g., Cefazolin, Cefuroxime)"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Serratia marcescens",
                aliases: ["serratia", "s. marcescens"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Cephalosporins (1st Gen)", "Nitrofurantoin", "Colistin", "Polymyxin B"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Proteus mirabilis",
                aliases: ["proteus mirabilis", "p. mirabilis"],
                resistances: ["Nitrofurantoin", "Colistin", "Polymyxin B", "Tetracyclines", "Tigecycline"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Proteus vulgaris",
                aliases: ["proteus vulgaris", "p. vulgaris"],
                resistances: ["Ampicillin", "Cephalosporins (1st & 2nd Gen)", "Nitrofurantoin", "Colistin", "Polymyxin B", "Tetracyclines", "Tigecycline", "Imipenem"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Morganella morganii",
                aliases: ["morganella", "m. morganii"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Cephalosporins (1st Gen)", "Nitrofurantoin", "Colistin", "Polymyxin B", "Tigecycline"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
             {
                organism: "Providencia spp.",
                aliases: ["providencia"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Cephalosporins (1st Gen)", "Nitrofurantoin", "Colistin", "Polymyxin B", "Tetracyclines", "Tigecycline"],
                generalNote: "As an Enterobacterales, also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Acinetobacter baumannii complex",
                aliases: ["acinetobacter", "a. baumannii"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Cephalosporins (1st & 2nd Gen)", "Ertapenem"],
                generalNote: "Also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Pseudomonas aeruginosa",
                aliases: ["pseudomonas", "p. aeruginosa"],
                resistances: ["Ampicillin", "Amoxicillin-clavulanate", "Ampicillin-sulbactam", "Cephalosporins (1st & 2nd Gen)", "Cefotaxime", "Ceftriaxone", "Ertapenem", "Trimethoprim-sulfamethoxazole", "Tetracyclines (except Minocycline)", "Chloramphenicol", "Fosfomycin"],
                generalNote: "Also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), Fusidic Acid, and Linezolid."
            },
            {
                organism: "Stenotrophomonas maltophilia",
                aliases: ["stenotrophomonas", "s. maltophilia"],
                resistances: ["Aminoglycosides", "Carbapenems (Imipenem, Meropenem, Ertapenem)", "Most β-lactams due to production of L1 (MBL) and L2 (cephalosporinase) β-lactamases."],
                generalNote: "Also intrinsically resistant to Penicillin G, Oxacillin, Macrolides, Lincosamides (Clindamycin), Streptogramins, Glycopeptides (Vancomycin), and Fusidic Acid."
            },
            {
                organism: "Enterococcus spp.",
                aliases: ["enterococcus", "vre"],
                resistances: ["Cephalosporins (all generations)", "Aminoglycosides (for monotherapy)", "Clindamycin", "Trimethoprim-sulfamethoxazole", "Fusidic Acid"],
                generalNote: "*E. faecium* is often intrinsically more resistant than *E. faecalis*, including to ampicillin and imipenem. *E. gallinarum/casseliflavus* have intrinsic low-level vancomycin resistance (VanC)."
            }
        ];

        // --- FIREBASE SETUP ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'clsi-breakpoint-navigator';

        let db, auth, userId;

        try {
            const app = initializeApp(firebaseConfig);
            db = getFirestore(app);
            auth = getAuth(app);

            onAuthStateChanged(auth, async (user) => {
                if (user) {
                    userId = user.uid;
                } else {
                    try {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                        userId = auth.currentUser?.uid || 'anonymous';
                    } catch (error) {
                        console.error("Authentication failed:", error);
                        userId = 'anonymous-error';
                    }
                }
                document.getElementById('user-id-display').textContent = userId;
            });
        } catch (e) {
            console.error("Firebase initialization failed. Comments will not be available.", e);
            document.getElementById('user-info').style.display = 'none';
        }


        // --- UI & SEARCH LOGIC ---
        const bugSearch = document.getElementById('bug-search');
        const drugSearch = document.getElementById('drug-search');
        const drugSearchContainer = document.getElementById('drug-search-container');
        const searchBtn = document.getElementById('search-btn');
        const resultsContainer = document.getElementById('results-container');
        const bugList = document.getElementById('bug-list');
        const drugList = document.getElementById('drug-list');
        const initialMessage = document.getElementById('initial-message');
        const modeRadios = document.querySelectorAll('input[name="search-mode"]');

        function populateDatalists() {
            const allBugs = new Set();
            const allDrugs = new Set();
            astData.forEach(group => {
                allBugs.add(group.organismGroup);
                group.aliases.forEach(alias => allBugs.add(alias));
                group.data.forEach(drug => allDrugs.add(drug.antimicrobial));
            });
            intrinsicResistanceData.forEach(bug => {
                allBugs.add(bug.organism);
                bug.aliases.forEach(alias => allBugs.add(alias));
            });
            
            bugList.innerHTML = [...allBugs].sort().map(bug => `<option value="${bug}"></option>`).join('');
            drugList.innerHTML = [...allDrugs].sort().map(drug => `<option value="${drug}"></option>`).join('');
        }
        
        function findBreakpointData(bugQuery, drugQuery) {
            bugQuery = bugQuery.toLowerCase().trim();
            drugQuery = drugQuery.toLowerCase().trim();
            
            // Prioritize exact species match first
            let organismGroup = astData.find(group => group.organismGroup.toLowerCase() === bugQuery);
            
            // If no exact match, try aliases
            if (!organismGroup) {
                organismGroup = astData.find(group => group.aliases.includes(bugQuery));
            }

            if (!organismGroup) return null;

            let drugData = organismGroup.data.find(drug => drug.antimicrobial.toLowerCase() === drugQuery);
            
            // If drug not found in specific group, check the general group (e.g., search "E. faecalis" for Linezolid, find it in "Enterococcus spp.")
            if (!drugData && organismGroup.organismGroup.includes('Enterococcus')) {
                const generalGroup = astData.find(g => g.organismGroup === 'Enterococcus spp.');
                if (generalGroup) {
                    const generalDrugData = generalGroup.data.find(d => d.antimicrobial.toLowerCase() === drugQuery);
                    if(generalDrugData) return { organismGroup, drugData: generalDrugData };
                }
            }
            
            if (!drugData) return null;
            
            return { organismGroup, drugData };
        }

        function findIntrinsicResistanceData(bugQuery) {
            bugQuery = bugQuery.toLowerCase().trim();
            return intrinsicResistanceData.find(bug => 
                bug.organism.toLowerCase() === bugQuery || bug.aliases.includes(bugQuery)
            );
        }
        
        function formatBreakpoint(bp) {
            if (typeof bp === 'object' && bp !== null) {
                let parts = [];
                if (bp.S !== 'N/A' && bp.S) parts.push(`<span class="font-semibold text-green-700">S:</span> ${bp.S}`);
                if (bp.SDD) parts.push(`<span class="font-semibold text-yellow-700">SDD:</span> ${bp.SDD}`);
                if (bp.I) parts.push(`<span class="font-semibold text-yellow-700">I:</span> ${bp.I}`);
                if (bp.R) parts.push(`<span class="font-semibold text-red-700">R:</span> ${bp.R}`);
                return parts.length > 0 ? parts.join(' | ') : 'N/A';
            }
            return bp || 'N/A';
        }

        function createAccordionSection(title, content, iconSvg) {
            if (!content || content.length === 0) return '';
            const contentHtml = `
                <ul class="list-none space-y-2 text-gray-700">
                    ${content.map(item => `
                        <li class="flex items-start">
                            <span class="text-indigo-500 mr-3 mt-1">&#9656;</span>
                            <span>${item}</span>
                        </li>`).join('')}
                </ul>`;

            return `
                <div class="border-t border-gray-200">
                    <div class="accordion-header flex justify-between items-center p-4">
                        <div class="flex items-center">
                            ${iconSvg}
                            <h3 class="font-semibold text-lg text-gray-800">${title}</h3>
                        </div>
                        <svg class="w-6 h-6 transform transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
                    </div>
                    <div class="accordion-content px-4">
                        ${contentHtml}
                    </div>
                </div>
            `;
        }

        function displayBreakpointResults(data) {
            if (initialMessage) initialMessage.style.display = 'none';
            resultsContainer.innerHTML = ''; 

            if (!data) {
                resultsContainer.innerHTML = `<div class="bg-white p-6 rounded-xl shadow-md text-center text-red-600 border border-red-200">No breakpoint data found for this combination. Please check your spelling or try another combination.</div>`;
                return;
            }

            const { organismGroup, drugData } = data;
            const comments = drugData.comments || {};

            const iconExam = `<svg class="icon text-indigo-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.246 18 16.5 18c-1.746 0-3.332.477-4.5 1.253"></path></svg>`;
            const iconMechanism = `<svg class="icon text-teal-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 3v2m6-2v2M9 19v2m6-2v2M5 9H3m2 6H3m18-6h-2m2 6h-2M12 6V3m0 18v-3"></path></svg>`;
            const iconNotes = `<svg class="icon text-amber-600" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z"></path></svg>`;
            
            const resultCard = `
                <div class="bg-white rounded-xl shadow-lg border border-gray-200 overflow-hidden">
                    <div class="p-6 bg-gray-50">
                        <h2 class="text-2xl font-bold mb-1 text-indigo-900">${drugData.antimicrobial}</h2>
                        <p class="text-lg text-gray-600">vs. ${organismGroup.organismGroup}</p>
                    </div>
                    
                    <div class="grid grid-cols-1 md:grid-cols-2 gap-0 border-t border-b border-gray-200">
                        <div class="p-4 border-b md:border-b-0 md:border-r border-gray-200">
                            <h3 class="font-semibold text-gray-700 mb-1">MIC Breakpoints (µg/mL)</h3>
                            <p class="text-gray-900 text-lg">${formatBreakpoint(drugData.mic)}</p>
                        </div>
                        <div class="p-4">
                            <h3 class="font-semibold text-gray-700 mb-1">Zone Diameter (mm)</h3>
                            <p class="text-gray-900 text-lg"><span class="text-sm font-medium">Disk: ${drugData.diskContent}</span> | ${formatBreakpoint(drugData.zoneDiameter)}</p>
                        </div>
                    </div>

                    ${createAccordionSection('Key Exam Points & Clinical Correlations', comments.exam, iconExam)}
                    ${createAccordionSection('Mechanism & Resistance Insights', comments.mechanism, iconMechanism)}

                    <!-- User Comments Section -->
                    <div class="border-t border-gray-200">
                        <div class="accordion-header flex justify-between items-center p-4">
                            <div class="flex items-center">
                                ${iconNotes}
                                <h3 class="font-semibold text-lg text-gray-800">My Notes / Institutional Comments</h3>
                            </div>
                            <svg class="w-6 h-6 transform transition-transform" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path></svg>
                        </div>
                        <div class="accordion-content px-4">
                            <div id="comments-list" class="space-y-3 mb-4 max-h-60 overflow-y-auto pr-2">
                               <p class="text-gray-500">Loading notes...</p>
                            </div>
                            <div class="mt-4">
                                <textarea id="comment-input" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500" rows="3" placeholder="Add your personal note or clinical pearl..."></textarea>
                                <button id="save-comment-btn" class="mt-2 bg-emerald-600 text-white font-semibold px-5 py-2 rounded-lg hover:bg-emerald-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-emerald-500 transition-all">Save Note</button>
                            </div>
                        </div>
                    </div>
                </div>
            `;
            resultsContainer.innerHTML = resultCard;
            
            document.querySelectorAll('.accordion-header').forEach(header => {
                header.addEventListener('click', () => {
                    const content = header.nextElementSibling;
                    const icon = header.querySelector('svg:last-child');
                    content.classList.toggle('open');
                    icon.classList.toggle('rotate-180');
                });
            });

            document.getElementById('save-comment-btn').addEventListener('click', () => saveComment(drugData.antimicrobial, organismGroup.organismGroup));
            fetchComments(drugData.antimicrobial, organismGroup.organismGroup);
        }

        function displayIntrinsicResistance(data) {
            if (initialMessage) initialMessage.style.display = 'none';
            resultsContainer.innerHTML = ''; 

            if (!data) {
                resultsContainer.innerHTML = `<div class="bg-white p-6 rounded-xl shadow-md text-center text-red-600 border border-red-200">No intrinsic resistance data found for this organism. Please check your spelling or try another combination.</div>`;
                return;
            }

            const resultCard = `
                <div class="bg-white rounded-xl shadow-lg border border-gray-200 overflow-hidden">
                    <div class="p-6 bg-gray-50">
                        <h2 class="text-2xl font-bold mb-1 text-indigo-900">Intrinsic Resistance Profile</h2>
                        <p class="text-lg text-gray-600">for ${data.organism}</p>
                    </div>
                    <div class="p-6 border-t border-gray-200">
                        <div class="bg-indigo-50 border border-indigo-200 text-indigo-800 p-4 rounded-lg mb-6">
                            <h3 class="font-semibold">What is Intrinsic Resistance?</h3>
                            <p class="text-sm mt-1">This is a predictable, innate resistance characteristic of all or almost all strains of a species. Testing these drugs is unnecessary and results should be reported as 'Resistant' if tested inadvertently.</p>
                        </div>
                        <h3 class="font-semibold text-lg text-gray-800 mb-3">Resistant to the following:</h3>
                        <ul class="list-disc list-inside space-y-2 text-gray-700 columns-1 sm:columns-2">
                            ${data.resistances.map(drug => `<li>${drug}</li>`).join('')}
                        </ul>
                        ${data.generalNote ? `<p class="mt-4 pt-4 border-t border-gray-200 text-sm text-gray-600">${data.generalNote}</p>` : ''}
                    </div>
                </div>
            `;
            resultsContainer.innerHTML = resultCard;
        }

        let unsubscribe;

        function fetchComments(drug, bug) {
            if (!db) {
                document.getElementById('comments-list').innerHTML = '<p class="text-red-500">Could not connect to database for notes.</p>';
                return;
            }
            if (unsubscribe) unsubscribe();

            const commentsCollection = collection(db, `artifacts/${appId}/public/data/comments`);
            const q = query(commentsCollection, where("drug", "==", drug), where("bug", "==", bug), orderBy("timestamp", "desc"));
            
            unsubscribe = onSnapshot(q, (querySnapshot) => {
                const commentsList = document.getElementById('comments-list');
                if (querySnapshot.empty) {
                    commentsList.innerHTML = '<p class="text-gray-500">No notes yet for this combination.</p>';
                } else {
                    commentsList.innerHTML = '';
                    querySnapshot.forEach((doc) => {
                        const commentData = doc.data();
                        const commentEl = document.createElement('div');
                        commentEl.className = 'bg-amber-50 p-3 rounded-lg border border-amber-200 text-sm';
                        const date = commentData.timestamp?.toDate ? commentData.timestamp.toDate().toLocaleString() : 'Just now';
                        commentEl.innerHTML = `
                            <p class="text-gray-800">${commentData.comment.replace(/\n/g, '<br>')}</p>
                            <p class="text-xs text-gray-500 mt-2">Added by ${commentData.userId.substring(0,8)}... on ${date}</p>
                        `;
                        commentsList.appendChild(commentEl);
                    });
                }
            }, (error) => {
                console.error("Error fetching comments: ", error);
                document.getElementById('comments-list').innerHTML = '<p class="text-red-500">Error loading notes.</p>';
            });
        }

        async function saveComment(drug, bug) {
            if (!db || !userId) {
                alert("Cannot save note. Database or user not available.");
                return;
            }
            const commentInput = document.getElementById('comment-input');
            const commentText = commentInput.value.trim();
            if (commentText) {
                try {
                    const commentsCollection = collection(db, `artifacts/${appId}/public/data/comments`);
                    await addDoc(commentsCollection, {
                        drug: drug,
                        bug: bug,
                        comment: commentText,
                        userId: userId,
                        timestamp: serverTimestamp()
                    });
                    commentInput.value = '';
                } catch (error) {
                    console.error("Error adding document: ", error);
                    alert("Failed to save note.");
                }
            }
        }

        function handleSearch() {
            const currentMode = document.querySelector('input[name="search-mode"]:checked').value;
            const bugQuery = bugSearch.value;

            if (!bugQuery) {
                resultsContainer.innerHTML = `<div class="bg-white p-6 rounded-xl shadow-md text-center text-yellow-600 border border-yellow-200">Please enter an organism to search.</div>`;
                return;
            }

            if (currentMode === 'breakpoint') {
                const drugQuery = drugSearch.value;
                if (!drugQuery) {
                    resultsContainer.innerHTML = `<div class="bg-white p-6 rounded-xl shadow-md text-center text-yellow-600 border border-yellow-200">Please enter a drug for breakpoint search.</div>`;
                    return;
                }
                const searchResult = findBreakpointData(bugQuery, drugQuery);
                displayBreakpointResults(searchResult);
            } else if (currentMode === 'intrinsic') {
                const searchResult = findIntrinsicResistanceData(bugQuery);
                displayIntrinsicResistance(searchResult);
            }
        }
        
        function updateSearchMode() {
            const currentMode = document.querySelector('input[name="search-mode"]:checked').value;
            if (currentMode === 'intrinsic') {
                drugSearchContainer.style.display = 'none';
                searchBtn.textContent = 'Show Intrinsic Resistance';
            } else {
                drugSearchContainer.style.display = 'block';
                searchBtn.textContent = 'Search Combination';
            }
            // Clear results when mode changes
            resultsContainer.innerHTML = '';
            if (initialMessage) initialMessage.style.display = 'block';
        }

        // Event Listeners
        searchBtn.addEventListener('click', handleSearch);
        bugSearch.addEventListener('keypress', (e) => e.key === 'Enter' && handleSearch());
        drugSearch.addEventListener('keypress', (e) => e.key === 'Enter' && handleSearch());
        modeRadios.forEach(radio => radio.addEventListener('change', updateSearchMode));

        // Initial setup
        populateDatalists();
        updateSearchMode();

    </script>
</body>
</html>
