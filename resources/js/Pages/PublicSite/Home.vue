<script setup>
import { Head, Link, usePage, router } from "@inertiajs/vue3";
import { ref, computed, watch, onMounted, onUnmounted } from "vue";
import PublicSiteLayout from "@/Layouts/PublicSite/PublicSiteLayout.vue";

const page = usePage();
const showroom = computed(() => ({
    name: "Apex Showroom",
    city: "Lahore",
    phone: "923001234567",
    address: "Main Auto Market, Gulberg III, Lahore",
    ...(page.props.institution || {}),
}));
const waLink = (t) => `https://wa.me/${showroom.value.phone.replace(/\D/g, "")}?text=${encodeURIComponent(t)}`;
const formatPkr = (n) => "PKR " + Number(n).toLocaleString("en-PK");
const u = (id, w = 800) => `https://images.unsplash.com/${id}?auto=format&fit=crop&w=${w}&q=80`;
const bg = (id, w = 1800) => ({ backgroundImage: `url(${u(id, w)})` });

/* ---------- Scroll reveal directive: v-reveal="delayMs" ---------- */
let io = null;
const vReveal = {
    mounted(el, b) {
        if (typeof IntersectionObserver === "undefined") {
            el.classList.add("rv-in");
            return;
        }
        if (!io) {
            io = new IntersectionObserver(
                (entries) => entries.forEach((e) => {
                    if (e.isIntersecting) {
                        e.target.classList.add("rv-in");
                        io.unobserve(e.target);
                    }
                }),
                { threshold: 0.08, rootMargin: "0px 0px -25px 0px" }
            );
        }
        el.classList.add("rv");
        el.style.setProperty("--d", (b.value || 0) + "ms");
        io.observe(el);
    },
};

/* ---------- Hero slideshow ---------- */
const heroSlides = [
    "photo-1503376780353-7e6692767b70",
    "photo-1617814076367-b759c7d7e738",
    "photo-1558981403-c5f9899a28bc",
    "photo-1586191582056-a60037a599ec",
].map((i) => u(i, 1920));
const slideLabels = ["Luxury Cars", "Sport Bikes", "Cruisers", "Loaders"];
const cur = ref(0);
let timer = null;
const goTo = (i) => {
    cur.value = i;
    start();
};
const start = () => {
    clearInterval(timer);
    if (typeof window !== "undefined" && window.matchMedia("(prefers-reduced-motion: reduce)").matches) return;
    timer = setInterval(() => {
        cur.value = (cur.value + 1) % heroSlides.length;
    }, 6000);
};

onMounted(() => {
    heroSlides.slice(1).forEach((s) => {
        const img = new Image();
        img.src = s;
    });
    start();
    window.addEventListener("keydown", onKey);
});

onUnmounted(() => {
    clearInterval(timer);
    window.removeEventListener("keydown", onKey);
    document.body.style.overflow = "";
});

/* ---------- Quick Search ---------- */
const search = ref({ category: "", budget: "", city: "" });
const budgets = [
    ["", "Any budget"],
    ["500000", "Under 5 lakh"],
    ["2000000", "Under 20 lakh"],
    ["6000000", "Under 60 lakh"],
    ["10000000", "Under 1 crore"],
];
const runSearch = () => {
    const params = Object.fromEntries(Object.entries(search.value).filter(([, v]) => v));
    router.get("/inventory", params);
};

/* ---------- Brands Marquee ---------- */
const brands = ["Toyota", "Honda", "Suzuki", "Yamaha", "KIA", "Hyundai", "Sazgar", "FAW", "Changan", "MG"];

/* ---------- Categories ---------- */
const groups = [
    {
        title: "Cars & SUVs",
        blurb: "Sedans, compact SUVs, hatchbacks and luxury imports",
        items: [
            { id: "sedans", title: "Executive Sedans", sub: "Corolla, Civic, Yaris, City", count: 28, img: "photo-1621007947382-bb3c3994e3fb", link: "/inventory?category=cars&body=sedan" },
            { id: "suvs", title: "4x4 & Compact SUVs", sub: "Fortuner, Sportage, Tucson", count: 18, img: "photo-1533473359331-0135ef1b58bf", link: "/inventory?category=cars&body=suv" },
            { id: "hatch", title: "City Hatchbacks", sub: "Alto, Cultus, WagonR, Swift", count: 22, img: "photo-1549399542-7e3f8b79c341", link: "/inventory?category=cars&body=hatchback" },
            { id: "luxury", title: "Luxury & Imports", sub: "Mercedes, BMW, Audi, Crown", count: 8, img: "photo-1552519507-da3b142c6e3d", link: "/inventory?category=cars&type=luxury" },
        ],
    },
    {
        title: "Motorcycles & Scooters",
        blurb: "Fuel-efficient commuters to high-performance superbikes",
        items: [
            { id: "commuter", title: "Commuter Bikes", sub: "CD 70, CG 125, Pridor", count: 35, img: "photo-1558981403-c5f9899a28bc", link: "/inventory?category=bikes&type=commuter" },
            { id: "sports", title: "Sports & Touring", sub: "YBR 125G, CB 150F, GSX", count: 19, img: "photo-1568772585407-9361f9bf3a87", link: "/inventory?category=bikes&type=sports" },
            { id: "heavy", title: "Superbikes & Cruisers", sub: "250cc to 600cc imports", count: 9, img: "photo-1609630875171-b1321377ee65", link: "/inventory?category=bikes&type=heavy" },
            { id: "ev", title: "Electric Scooters", sub: "Lithium-ion powered rides", count: 12, img: "photo-1571068316344-75bc76f77890", link: "/inventory?category=bikes&type=electric" },
        ],
    },
    {
        title: "Loaders & Commercial",
        blurb: "Heavy-duty loaders, hydraulic tippers, pickups and delivery vans",
        items: [
            { id: "cargo", title: "200cc Cargo Loaders", sub: "Sazgar, New Asia, Road Prince", count: 24, img: "photo-1519003722824-194d4455a60c", link: "/inventory?category=loaders&type=200cc" },
            { id: "tipper", title: "Hydraulic Tippers", sub: "Sand and construction haulers", count: 11, img: "photo-1586191582056-a60037a599ec", link: "/inventory?category=loaders&type=hydraulic" },
            { id: "pickup", title: "Mini Pickups", sub: "FAW Carrier, Ravi, Mega", count: 14, img: "photo-1601584115197-04ecc0da31d7", link: "/inventory?category=loaders&type=mini-truck" },
            { id: "vans", title: "Commercial Delivery Vans", sub: "Suzuki Bolan, Hiace Cargo", count: 16, img: "photo-1559416523-140ddc3d238c", link: "/inventory?category=loaders&type=vans" },
        ],
    },
];

/* ---------- Featured Vehicles ---------- */
const vehicles = [
    { id: 1, title: "Toyota Fortuner Legender 2.8 Sigma", category: "cars", year: 2024, price: 18900000, condition: "Used", mileage: "8,500 km", engine: "2800 cc", transmission: "Automatic", fuel: "Diesel", location: "Lahore", badge: "Verified file", img: "photo-1533473359331-0135ef1b58bf" },
    { id: 2, title: "Honda Civic RS Turbo 1.5 Oriel", category: "cars", year: 2023, price: 9250000, condition: "Used", mileage: "14,200 km", engine: "1500 cc", transmission: "CVT", fuel: "Petrol", location: "Islamabad", badge: "First owner", img: "photo-1605559424843-9e4c228bf1c2" },
    { id: 3, title: "Toyota Corolla Altis Grande 1.8", category: "cars", year: 2022, price: 5850000, condition: "Used", mileage: "24,000 km", engine: "1800 cc", transmission: "CVT", fuel: "Petrol", location: "Lahore", badge: "Total genuine", img: "photo-1621007947382-bb3c3994e3fb" },
    { id: 4, title: "Yamaha YZF-R3 Super Sport", category: "bikes", year: 2024, price: 2450000, condition: "Used", mileage: "3,100 km", engine: "321 cc", transmission: "6-speed", fuel: "Petrol", location: "Karachi", badge: "Import unit", img: "photo-1568772585407-9361f9bf3a87" },
    { id: 5, title: "Honda CB 150F Special Edition", category: "bikes", year: 2024, price: 520000, condition: "New", mileage: "0 km", engine: "150 cc", transmission: "5-speed", fuel: "Petrol", location: "Lahore", badge: "Brand new", img: "photo-1558981403-c5f9899a28bc" },
    { id: 6, title: "Yamaha YBR 125G Trail Bronze", category: "bikes", year: 2023, price: 390000, condition: "Used", mileage: "6,200 km", engine: "125 cc", transmission: "5-speed", fuel: "Petrol", location: "Rawalpindi", badge: "Single hand", img: "photo-1609630875171-b1321377ee65" },
    { id: 7, title: "Sazgar 200cc Heavy Duty Cargo Loader", category: "loaders", year: 2024, price: 685000, condition: "New", mileage: "0 km", engine: "200 cc", transmission: "4-speed + rev", fuel: "Petrol/CNG", location: "Lahore", badge: "Factory ready", img: "photo-1519003722824-194d4455a60c" },
    { id: 8, title: "New Asia 150cc Hydraulic Tipper", category: "loaders", year: 2024, price: 590000, condition: "New", mileage: "0 km", engine: "150 cc", transmission: "Heavy gear", fuel: "Petrol", location: "Faisalabad", badge: "Hydraulic dump", img: "photo-1586191582056-a60037a599ec" },
    { id: 9, title: "FAW Tiger V 4x2 Mini Cargo Truck", category: "loaders", year: 2022, price: 3650000, condition: "Used", mileage: "36,000 km", engine: "2771 cc", transmission: "Manual", fuel: "Diesel", location: "Lahore", badge: "Commercial pass", img: "photo-1601584115197-04ecc0da31d7" },
];
const tabs = [
    { id: "all", label: "All Vehicles" },
    { id: "cars", label: "Cars" },
    { id: "bikes", label: "Bikes" },
    { id: "loaders", label: "Loaders" },
];
const activeTab = ref("all");
const shown = computed(() => (activeTab.value === "all" ? vehicles : vehicles.filter((v) => v.category === activeTab.value)));
const wishlist = ref(new Set());
const toggleWishlist = (id) => {
    const n = new Set(wishlist.value);
    n.has(id) ? n.delete(id) : n.add(id);
    wishlist.value = n;
};

/* ---------- Content: Stats, Pillars & Steps ---------- */
const stats = [
    ["150+", "Vehicles in stock"],
    ["3", "Categories, one roof"],
    ["100%", "Documents checked"],
    ["Same day", "Inquiry reply"],
];

const pillars = [
    ["Cars, bikes and loaders", "One premier showroom for family cars, personal rides, and commercial utility vehicles.", "M3 13h2l1-4h12l1 4h2M7 17a2 2 0 100-4 2 2 0 000 4zm10 0a2 2 0 100-4 2 2 0 000 4z"],
    ["Clear asking prices", "All prices and paperwork details are published upfront. No surprise fees added later.", "M12 8c-1.7 0-3 .9-3 2s1.3 2 3 2 3 .9 3 2-1.3 2-3 2m0-8V6m0 12v-2"],
    ["Honest specifications", "Engine capacity, gearbox health, body condition, and genuine mileage reported honestly.", "M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z"],
    ["Fast WhatsApp response", "Inspect vehicles on-site or request video walkthroughs and invoices directly on WhatsApp.", "M8 10h8M8 14h5m-9 6l2-4H5a2 2 0 01-2-2V6a2 2 0 012-2h14a2 2 0 012 2v8a2 2 0 01-2 2h-7l-4 4z"],
    ["Biometric & transfer help", "Our dedicated documentation team handles biometric verification, excise files, and token tax.", "M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.6L19 9.4V19a2 2 0 01-2 2z"],
    ["Commercial fleet support", "Bulk loader inquiries, corporate transport orders, and custom financing guidance.", "M17 20h5v-2a3 3 0 00-5.4-1.8M17 20H7m10 0v-2c0-.7-.1-1.3-.4-1.8M7 20H2v-2a3 3 0 015.4-1.8M7 20v-2c0-.7.1-1.3.4-1.8m0 0a5 5 0 019.2 0M15 7a3 3 0 11-6 0 3 3 0 016 0z"],
];

const steps = [
    ["Share vehicle details", "Submit make, model, year, and registration information online or via WhatsApp."],
    ["We inspect and value it", "Bring your vehicle for a quick physical inspection or send photos for an immediate estimate."],
    ["We find buyers & close deal", "We manage inquiries, secure payments, and finalize the biometric transfer with peace of mind."],
];

/* ---------- Modals ---------- */
const quickView = ref(null);
const sellOpen = ref(false);
const blank = () => ({ name: "", phone: "", category: "cars", vehicle: "", price: "" });
const sellForm = ref(blank());
const sent = ref(false);

watch([quickView, sellOpen], ([a, b]) => {
    document.body.style.overflow = a || b ? "hidden" : "";
});

const onKey = (e) => {
    if (e.key === "Escape") {
        quickView.value = null;
        sellOpen.value = false;
    }
};

const submitSell = () => {
    sent.value = true;
    setTimeout(() => {
        sent.value = false;
        sellOpen.value = false;
        sellForm.value = blank();
    }, 2200);
};

/* ---------- Style Constants ---------- */
const field = "w-full rounded-xl border border-slate-300 bg-white px-3.5 py-3 text-base sm:text-sm text-slate-900 placeholder:text-slate-400 transition focus:border-amber-500 focus:outline-none focus:ring-4 focus:ring-amber-500/20";
const btnGold = "btn-gold inline-flex items-center justify-center gap-2 rounded-xl bg-gradient-to-r from-amber-400 via-amber-400 to-amber-500 px-5 sm:px-6 py-3 text-sm font-bold text-slate-950 shadow-md shadow-amber-500/25 transition duration-300 hover:-translate-y-0.5 hover:shadow-xl hover:shadow-amber-500/40 active:translate-y-0 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-amber-400 cursor-pointer";
const btnGlass = "inline-flex items-center justify-center gap-2 rounded-xl border border-white/30 bg-white/10 px-5 sm:px-6 py-3 text-sm font-bold text-white backdrop-blur-md transition duration-300 hover:-translate-y-0.5 hover:bg-white hover:text-slate-950 active:translate-y-0 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-white cursor-pointer";
</script>

<template>
    <Head :title="`${showroom.name} - Luxury Cars, Bikes & Commercial Loaders`" />
    <PublicSiteLayout>
        <div class="overflow-x-hidden bg-slate-50 text-slate-900">
            <!-- ================= HERO SECTION ================= -->
            <section class="relative isolate flex min-h-[min(620px,100svh)] sm:min-h-[640px] lg:min-h-[calc(100svh-5.25rem)] xl:min-h-[720px] xl:max-h-[880px] flex-col justify-center bg-[#06101e] text-white">
                <!-- Background Slideshow -->
                <div class="absolute inset-0 -z-10 overflow-hidden">
                    <img
                        v-for="(s, i) in heroSlides"
                        :key="s"
                        :src="s"
                        alt="Hero vehicle background"
                        :loading="i === 0 ? 'eager' : 'lazy'"
                        class="hero-slide absolute inset-0 h-full w-full object-cover"
                        :class="cur === i ? 'is-on' : ''"
                    />
                    <!-- Gradients for pristine contrast across all viewports -->
                    <div class="absolute inset-0 bg-gradient-to-r from-[#06101e]/95 via-[#06101e]/75 to-[#06101e]/40 lg:to-transparent"></div>
                    <div class="absolute inset-0 bg-gradient-to-t from-[#06101e] via-[#06101e]/30 to-[#06101e]/70"></div>
                    <div class="glow absolute -left-20 top-1/4 h-72 w-72 rounded-full bg-amber-500/15 blur-[120px] pointer-events-none"></div>
                </div>

                <!-- Hero Content Container -->
                <div class="mx-auto w-full max-w-7xl px-4 py-8 sm:px-6 sm:py-12 md:py-14 lg:px-8 lg:py-16">
                    <div class="max-w-3xl">
                        <!-- Location & Showroom Pill Badge -->
                        <div class="hero-in inline-flex items-center gap-2 rounded-full border border-amber-400/40 bg-amber-400/10 px-3 sm:px-3.5 py-1.5 text-xs font-semibold text-amber-200 backdrop-blur-md" style="--i: 0">
                            <span class="relative flex h-2 w-2">
                                <span class="absolute inline-flex h-full w-full animate-ping rounded-full bg-amber-400 opacity-75"></span>
                                <span class="relative inline-flex h-2 w-2 rounded-full bg-amber-400"></span>
                            </span>
                            <span>{{ showroom.name }} · {{ showroom.city }}</span>
                        </div>

                        <!-- Main Headline -->
                        <h1 class="hero-in mt-4 text-3xl font-black leading-[1.08] tracking-tight min-[420px]:text-4xl sm:text-5xl lg:text-6xl xl:text-[3.5rem]" style="--i: 1">
                            Excellence in Every Deal.
                            <span class="shimmer block mt-1">Confidence in Every Drive.</span>
                        </h1>

                        <!-- Subtitle -->
                       <p class="hero-in mt-4 max-w-2xl text-sm leading-relaxed text-slate-200/95 sm:mt-5 sm:text-base md:text-lg" style="--i: 2">
                            Cars, bikes, loaders and commercial vehicles all under one roof. Compare verified asking prices, inspect documentation, and drive away with absolute peace of mind.
                            Find the right vehicle at the right price with a smooth, transparent, and hassle-free buying experience.
                        </p>

                        <!-- Primary CTA Actions -->
                        <div class="hero-in mt-6 flex flex-col gap-3 min-[420px]:flex-row sm:mt-8 sm:gap-4" style="--i: 3">
                            <Link href="/inventory" :class="btnGold">
                                <span>Browse inventory</span>
                                <svg class="h-4 w-4 transition-transform group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2.2" d="M14 5l7 7-7 7M3 12h18" /></svg>
                            </Link>
                            <button type="button" @click="sellOpen = true" :class="btnGlass">
                                <svg class="h-4 w-4 text-amber-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 6v6m0 0v6m0-6h6m-6 0H6" /></svg>
                                <span>Sell your vehicle</span>
                            </button>
                        </div>
                    </div>
                </div>
            </section>

            <!-- ================= BRAND MARQUEE ================= -->
            <section class="overflow-hidden border-b border-slate-200 bg-white py-3.5 sm:py-4" aria-label="Brands we deal in">
                <div class="marquee flex w-max items-center gap-10 whitespace-nowrap text-base font-bold text-slate-400 sm:gap-14 sm:text-lg lg:text-xl">
                    <template v-for="(b, i) in [...brands, ...brands, ...brands, ...brands]" :key="i">
                        <span class="transition-colors hover:text-amber-500 cursor-default">{{ b }}</span>
                        <span class="text-amber-400/40 text-xs sm:text-sm" aria-hidden="true">•</span>
                    </template>
                </div>
            </section>

            <!-- ================= CATEGORIES SECTION ================= -->
            <section class="relative isolate py-12 sm:py-16 md:py-20 lg:py-24">
                <div class="absolute inset-0 -z-10 bg-cover bg-center lg:bg-fixed" :style="bg('photo-1492144534655-ae79c964c9d7')"></div>
                <div class="absolute inset-0 -z-10 bg-gradient-to-b from-[#06101e] via-[#0a1a30]/95 to-[#06101e]"></div>

                <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
                    <!-- Section Header -->
                    <div v-reveal class="mb-8 max-w-2xl text-white sm:mb-12">
                        <div class="inline-flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-amber-400 mb-2">Showroom Directory</div>
                        <h2 class="text-2xl font-black tracking-tight text-white min-[400px]:text-3xl sm:text-4xl lg:text-5xl">Shop by category</h2>
                        <p class="mt-2 text-sm text-slate-300 sm:mt-3 sm:text-base">Pick the type of vehicle you need and browse available inventory ready for test drive.</p>
                    </div>

                    <!-- Category Groups -->
                    <div class="space-y-10 sm:space-y-12 lg:space-y-14">
                        <div v-for="g in groups" :key="g.title">
                            <!-- Group Title & Blurb -->
                            <div v-reveal class="mb-4 flex flex-wrap items-baseline justify-between gap-x-4 gap-y-1 border-b border-white/15 pb-2.5 text-white sm:mb-5">
                                <h3 class="text-lg font-bold text-amber-300 sm:text-xl">{{ g.title }}</h3>
                                <p class="text-xs text-slate-300 sm:text-sm">{{ g.blurb }}</p>
                            </div>

                            <!-- Group Cards Grid -->
                            <div class="grid grid-cols-1 gap-4 min-[480px]:grid-cols-2 lg:grid-cols-4 lg:gap-6">
                                <Link
                                    v-for="(c, i) in g.items"
                                    :key="c.id"
                                    :href="c.link"
                                    v-reveal="i * 80"
                                    class="cat-card group relative block aspect-[16/10] min-[480px]:aspect-[4/3] lg:aspect-[3/4] overflow-hidden rounded-2xl bg-slate-900 ring-1 ring-white/10 transition duration-500 hover:-translate-y-1.5 hover:ring-amber-400/80 hover:shadow-2xl hover:shadow-amber-500/20 focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-amber-400"
                                >
                                    <img :src="u(c.img)" :alt="c.title" loading="lazy" class="absolute inset-0 h-full w-full object-cover transition duration-700 group-hover:scale-110" />
                                    <div class="absolute inset-0 bg-gradient-to-t from-[#06101e] via-[#06101e]/40 to-transparent transition-opacity duration-500 group-hover:from-[#06101e]/95"></div>
                                    <span class="absolute right-3 top-3 rounded-full bg-amber-400 px-2.5 py-1 text-xs font-bold text-slate-950 shadow-md">
                                        {{ c.count }} available
                                    </span>
                                    <div class="absolute inset-x-0 bottom-0 p-4 sm:p-5 text-white">
                                        <h4 class="text-base font-bold leading-snug sm:text-lg">{{ c.title }}</h4>
                                        <p class="mt-1 text-xs text-slate-300 sm:text-sm line-clamp-1">{{ c.sub }}</p>
                                        <span class="mt-2.5 inline-flex items-center gap-1.5 text-xs font-bold text-amber-300 sm:mt-3 sm:text-sm">
                                            <span>View stock</span>
                                            <svg class="h-4 w-4 transition-transform group-hover:translate-x-1" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7-7 7M3 12h18" /></svg>
                                        </span>
                                    </div>
                                </Link>
                            </div>
                        </div>
                    </div>
                </div>
            </section>

            <!-- ================= FEATURED VEHICLES SECTION ================= -->
            <section class="relative isolate bg-gradient-to-b from-slate-50 via-sky-50/50 to-slate-100 py-12 sm:py-16 md:py-20 lg:py-24">
                <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
                    <!-- Section Header & Filter Tabs -->
                    <div v-reveal class="mb-8 flex flex-col gap-4 sm:mb-10 sm:gap-6 md:flex-row md:items-end md:justify-between">
                        <div class="max-w-xl">
                            <div class="inline-flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-amber-600 mb-1.5">Showroom Highlights</div>
                            <h2 class="text-2xl font-black tracking-tight text-slate-900 min-[400px]:text-3xl sm:text-4xl lg:text-5xl">Featured vehicles</h2>
                            <p class="mt-2 text-sm text-slate-600 sm:text-base">Hand-picked units with verified documents, ready for immediate inspection.</p>
                        </div>
                        <div role="tablist" class="flex w-full overflow-x-auto rounded-xl bg-white p-1 shadow-sm ring-1 ring-slate-200 sm:w-auto no-scrollbar">
                            <button
                                v-for="t in tabs"
                                :key="t.id"
                                role="tab"
                                :aria-selected="activeTab === t.id"
                                @click="activeTab = t.id"
                                :class="[
                                    'flex-1 whitespace-nowrap rounded-lg px-3.5 py-2 text-xs font-bold transition duration-200 sm:flex-initial sm:px-5 sm:text-sm cursor-pointer',
                                    activeTab === t.id
                                        ? 'bg-[#0a1a30] text-amber-300 shadow-sm'
                                        : 'text-slate-600 hover:bg-slate-100'
                                ]"
                            >
                                {{ t.label }}
                            </button>
                        </div>
                    </div>

                    <!-- Vehicles Grid -->
                    <TransitionGroup name="card" tag="div" class="grid grid-cols-1 gap-5 sm:grid-cols-2 lg:grid-cols-3 lg:gap-6">
                        <article
                            v-for="v in shown"
                            :key="v.id"
                            class="sheen group flex flex-col overflow-hidden rounded-2xl bg-white ring-1 ring-slate-200 transition duration-300 hover:-translate-y-1.5 hover:shadow-2xl hover:shadow-sky-950/10 hover:ring-amber-400"
                        >
                            <!-- Card Image Box -->
                            <div class="relative aspect-[16/10] overflow-hidden bg-slate-100">
                                <img :src="u(v.img)" :alt="v.title" loading="lazy" class="vehicle-card-img h-full w-full object-cover transition duration-700 group-hover:scale-108" />
                                <div class="absolute inset-0 bg-gradient-to-t from-black/60 via-transparent to-transparent"></div>
                                <span class="absolute left-3 top-3 rounded-md bg-amber-400 px-2.5 py-1 text-xs font-bold text-slate-950 shadow-md">
                                    {{ v.badge }}
                                </span>
                                <button
                                    type="button"
                                    @click="toggleWishlist(v.id)"
                                    :aria-pressed="wishlist.has(v.id)"
                                    aria-label="Save vehicle to favorites"
                                    :class="[
                                        'absolute right-3 top-3 flex h-9 w-9 items-center justify-center rounded-full bg-white/95 shadow-md transition duration-300 hover:scale-110 cursor-pointer',
                                        wishlist.has(v.id) ? 'text-rose-500' : 'text-slate-500'
                                    ]"
                                >
                                    <svg class="h-4 w-4" :fill="wishlist.has(v.id) ? 'currentColor' : 'none'" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z" /></svg>
                                </button>
                                <span class="absolute bottom-3 left-3 rounded-md bg-black/65 px-2.5 py-1 text-xs font-semibold text-white backdrop-blur-sm">
                                    {{ v.condition }} · {{ v.year }}
                                </span>
                            </div>

                            <!-- Card Body Content -->
                            <div class="flex flex-1 flex-col p-4 sm:p-5">
                                <p class="flex items-center gap-1.5 text-xs font-semibold text-slate-500">
                                    <svg class="h-3.5 w-3.5 text-amber-500 shrink-0" fill="currentColor" viewBox="0 0 20 20"><path fill-rule="evenodd" d="M10 18s6-5.2 6-10a6 6 0 10-12 0c0 4.8 6 10 6 10zm0-7.5a2.5 2.5 0 110-5 2.5 2.5 0 010 5z" clip-rule="evenodd" /></svg>
                                    <span>{{ v.location }}</span>
                                </p>

                                <!-- Title: line-clamp and min-h ensures 100% uniform alignment in every row -->
                                <h3 class="mt-2 text-base font-bold leading-snug text-slate-900 line-clamp-2 min-h-[2.75rem] sm:text-lg sm:min-h-[3.25rem] transition-colors group-hover:text-amber-600">
                                    {{ v.title }}
                                </h3>

                                <p class="mt-2 text-xl font-black text-[#0a1a30] sm:text-2xl tracking-tight">
                                    {{ formatPkr(v.price) }}
                                </p>

                                <!-- Specifications Matrix -->
                                <dl class="mt-3.5 grid grid-cols-2 gap-2 rounded-xl border border-slate-100 bg-slate-50 p-3 text-xs sm:mt-4 sm:gap-2.5 sm:p-3.5 sm:text-sm">
                                    <div>
                                        <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Engine</dt>
                                        <dd class="font-bold text-slate-800 truncate">{{ v.engine }}</dd>
                                    </div>
                                    <div>
                                        <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Mileage</dt>
                                        <dd class="font-bold text-slate-800 truncate">{{ v.mileage }}</dd>
                                    </div>
                                    <div>
                                        <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Gearbox</dt>
                                        <dd class="font-bold text-slate-800 truncate">{{ v.transmission }}</dd>
                                    </div>
                                    <div>
                                        <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Fuel</dt>
                                        <dd class="font-bold text-slate-800 truncate">{{ v.fuel }}</dd>
                                    </div>
                                </dl>

                                <!-- Action Buttons: Aligned at bottom of every card -->
                                <div class="mt-auto grid grid-cols-2 gap-2.5 pt-4 sm:gap-3 sm:pt-5">
                                    <button
                                        type="button"
                                        @click="quickView = v"
                                        class="inline-flex items-center justify-center rounded-xl border-2 border-slate-200 bg-white px-3 py-2.5 text-xs font-bold text-slate-800 transition duration-300 hover:border-[#0a1a30] hover:bg-[#0a1a30] hover:text-white sm:text-sm cursor-pointer"
                                    >
                                        Details
                                    </button>
                                    <a
                                        :href="waLink(`Assalam-o-Alaikum, I am interested in the ${v.title} (${formatPkr(v.price)})`)"
                                        target="_blank"
                                        rel="noopener"
                                        class="inline-flex items-center justify-center gap-1.5 rounded-xl bg-gradient-to-r from-amber-400 to-amber-500 px-3 py-2.5 text-xs font-bold text-slate-950 shadow-md shadow-amber-500/20 transition duration-300 hover:-translate-y-0.5 hover:shadow-lg hover:shadow-amber-500/35 active:translate-y-0 sm:text-sm cursor-pointer"
                                    >
                                        <svg class="h-4 w-4 shrink-0" fill="currentColor" viewBox="0 0 24 24"><path d="M12.031 6.172c-3.181 0-5.767 2.586-5.768 5.766-.001 1.298.38 2.27 1.019 3.287l-.582 2.128 2.182-.573c.978.58 1.911.928 3.145.929 3.178 0 5.767-2.587 5.768-5.766.001-3.187-2.575-5.77-5.764-5.771zm3.392 8.244c-.144.405-.837.774-1.17.824-.299.045-.677.063-1.092-.069-.252-.08-.575-.187-.988-.365-1.739-.751-2.874-2.502-2.961-2.617-.087-.116-.708-.94-.708-1.793s.448-1.273.607-1.446c.159-.173.346-.217.462-.217l.332.007c.106.005.249-.04.39.298.144.347.491 1.2.534 1.287.043.087.072.188.014.304-.058.116-.087.188-.173.289l-.26.304c-.087.086-.177.18-.076.354.101.174.449.741.964 1.201.662.591 1.221.774 1.394.86.174.086.275.072.376-.043.101-.116.433-.506.549-.68.116-.173.231-.145.39-.087s1.011.477 1.184.564.289.13.332.202c.043.073.043.419-.101.824z" /></svg>
                                        <span>WhatsApp</span>
                                    </a>
                                </div>
                            </div>
                        </article>
                    </TransitionGroup>

                    <!-- View All Vehicles CTA -->
                    <div v-reveal class="mt-10 sm:mt-12 text-center">
                        <Link
                            href="/inventory"
                            class="inline-flex items-center justify-center gap-2 rounded-xl bg-[#0a1a30] px-7 py-3.5 text-sm font-bold text-white shadow-lg shadow-sky-950/20 transition duration-300 hover:-translate-y-0.5 hover:bg-amber-500 hover:text-slate-950 hover:shadow-xl cursor-pointer"
                        >
                            <span>View all 150+ vehicles</span>
                            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M14 5l7 7-7 7M3 12h18" /></svg>
                        </Link>
                    </div>
                </div>
            </section>

            <!-- ================= SELL YOUR VEHICLE SECTION ================= -->
            <section class="relative isolate py-12 sm:py-16 md:py-20 lg:py-24">
                <div class="absolute inset-0 -z-10 bg-cover bg-center lg:bg-fixed" :style="bg('photo-1563720223185-11003d516935')"></div>
                <!-- Dynamic responsive gradient for mobile & desktop legibility -->
                <div class="absolute inset-0 -z-10 bg-gradient-to-b from-[#06101e]/95 via-[#06101e]/90 to-[#06101e]/80 lg:bg-gradient-to-r lg:from-[#06101e]/95 lg:via-[#06101e]/85 lg:to-[#06101e]/65"></div>

                <div class="mx-auto grid max-w-7xl items-center gap-8 px-4 sm:px-6 lg:grid-cols-2 lg:gap-16 lg:px-8">
                    <!-- Left: Description and Trigger -->
                    <div v-reveal class="text-white">
                        <div class="inline-flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-amber-400 mb-2">Sell or Exchange</div>
                        <h2 class="text-2xl font-black tracking-tight text-white min-[400px]:text-3xl sm:text-4xl lg:text-5xl">
                            Selling a vehicle? We find the verified buyer.
                        </h2>
                        <p class="mt-3 max-w-xl text-sm leading-relaxed text-slate-300 sm:mt-4 sm:text-base">
                            Put your car, bike, or commercial loader in front of hundreds of genuine walk-in buyers and verified automotive investors every week. We handle inquiries, test drives, and biometric transfer.
                        </p>
                        <div class="mt-6 sm:mt-8">
                            <button type="button" @click="sellOpen = true" :class="[btnGold, 'w-full sm:w-auto']">
                                <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12h6m-6 4h6m2 5H7a2 2 0 01-2-2V5a2 2 0 012-2h5.6L19 9.4V19a2 2 0 01-2 2z" /></svg>
                                <span>Request a free valuation</span>
                            </button>
                        </div>
                    </div>

                    <!-- Right: Step-by-Step Workflow Cards -->
                    <ol class="space-y-3 sm:space-y-4">
                        <li
                            v-for="(s, i) in steps"
                            :key="s[0]"
                            v-reveal="i * 120"
                            class="group flex items-start gap-3.5 rounded-2xl border border-white/15 bg-white/10 p-4 text-white backdrop-blur-md transition duration-300 hover:translate-x-1 hover:border-amber-400/60 hover:bg-white/15 sm:gap-4 sm:p-5"
                        >
                            <span class="flex h-9 w-9 sm:h-10 sm:w-10 shrink-0 items-center justify-center rounded-full bg-amber-400 text-sm sm:text-base font-black text-slate-950 shadow-md transition group-hover:scale-110">
                                {{ i + 1 }}
                            </span>
                            <div>
                                <h3 class="text-base font-bold text-white sm:text-lg">{{ s[0] }}</h3>
                                <p class="mt-1 text-xs text-slate-300 sm:text-sm leading-relaxed">{{ s[1] }}</p>
                            </div>
                        </li>
                    </ol>
                </div>
            </section>

            <!-- ================= WHY BUYERS CHOOSE US SECTION ================= -->
            <section class="bg-white py-12 sm:py-16 md:py-20 lg:py-24">
                <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
                    <!-- Section Header -->
                    <div v-reveal class="mb-8 max-w-2xl sm:mb-12">
                        <div class="inline-flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-amber-600 mb-1.5">Trust & Integrity</div>
                        <h2 class="text-2xl font-black tracking-tight text-slate-900 min-[400px]:text-3xl sm:text-4xl lg:text-5xl">
                            Why buyers choose {{ showroom.name }}
                        </h2>
                        <p class="mt-2 text-sm text-slate-600 sm:mt-3 sm:text-base">Transparent pricing, verified legal paperwork, and experienced advisors dedicated to helping you make the right choice.</p>
                    </div>

                    <!-- Pillars Grid (divisible by 2 on tablet, 3 on desktop) -->
                    <div class="grid gap-4 sm:gap-5 md:grid-cols-2 lg:grid-cols-3">
                        <div
                            v-for="(p, i) in pillars"
                            :key="p[0]"
                            v-reveal="(i % 3) * 100"
                            class="group flex flex-col rounded-2xl border border-slate-200 bg-slate-50/80 p-5 transition duration-300 hover:-translate-y-1.5 hover:border-amber-400 hover:bg-white hover:shadow-xl hover:shadow-amber-500/10 sm:p-6"
                        >
                            <div class="flex h-11 w-11 sm:h-12 sm:w-12 items-center justify-center rounded-xl bg-[#0a1a30] text-amber-300 transition duration-300 group-hover:bg-amber-400 group-hover:text-slate-950 group-hover:rotate-6">
                                <svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="1.8" :d="p[2]" /></svg>
                            </div>
                            <h3 class="mt-4 text-base font-bold text-slate-900 sm:text-lg">{{ p[0] }}</h3>
                            <p class="mt-1.5 text-xs sm:text-sm leading-relaxed text-slate-600 flex-1">{{ p[1] }}</p>
                        </div>
                    </div>
                </div>
            </section>

            <!-- ================= CTA / SHOWROOM VISIT SECTION ================= -->
            <section class="relative isolate py-12 sm:py-16 md:py-20 lg:py-24">
                <div class="absolute inset-0 -z-10 bg-cover bg-center lg:bg-fixed" :style="bg('photo-1580273916550-e323be2ae537')"></div>
                <div class="absolute inset-0 -z-10 bg-gradient-to-r from-[#06101e]/95 via-[#06101e]/85 to-[#06101e]/70"></div>

                <div v-reveal class="mx-auto flex max-w-7xl flex-col items-start gap-6 px-4 sm:px-6 lg:flex-row lg:items-center lg:justify-between lg:gap-8 lg:px-8">
                    <div class="max-w-2xl text-white">
                        <div class="inline-flex items-center gap-2 text-xs font-bold uppercase tracking-wider text-amber-400 mb-2">Visit Our Showroom</div>
                        <h2 class="text-2xl font-black tracking-tight text-white min-[400px]:text-3xl sm:text-4xl lg:text-[2.6rem]">
                            Visit us and experience the vehicle in person.
                        </h2>
                        <p class="mt-2.5 flex items-start gap-2 text-sm text-slate-300 sm:mt-3 sm:text-base">
                            <svg class="h-5 w-5 shrink-0 text-amber-400 mt-0.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" /><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" /></svg>
                            <span>{{ showroom.address }}</span>
                        </p>
                    </div>

                    <div class="flex w-full flex-col gap-3 sm:w-auto sm:flex-row">
                        <a :href="waLink('Assalam-o-Alaikum, I would like to visit the showroom.')" target="_blank" rel="noopener" :class="btnGold">
                            <svg class="h-4 w-4 shrink-0" fill="currentColor" viewBox="0 0 24 24"><path d="M12.031 6.172c-3.181 0-5.767 2.586-5.768 5.766-.001 1.298.38 2.27 1.019 3.287l-.582 2.128 2.182-.573c.978.58 1.911.928 3.145.929 3.178 0 5.767-2.587 5.768-5.766.001-3.187-2.575-5.77-5.764-5.771zm3.392 8.244c-.144.405-.837.774-1.17.824-.299.045-.677.063-1.092-.069-.252-.08-.575-.187-.988-.365-1.739-.751-2.874-2.502-2.961-2.617-.087-.116-.708-.94-.708-1.793s.448-1.273.607-1.446c.159-.173.346-.217.462-.217l.332.007c.106.005.249-.04.39.298.144.347.491 1.2.534 1.287.043.087.072.188.014.304-.058.116-.087.188-.173.289l-.26.304c-.087.086-.177.18-.076.354.101.174.449.741.964 1.201.662.591 1.221.774 1.394.86.174.086.275.072.376-.043.101-.116.433-.506.549-.68.116-.173.231-.145.39-.087s1.011.477 1.184.564.289.13.332.202c.043.073.043.419-.101.824z" /></svg>
                            <span>Chat on WhatsApp</span>
                        </a>
                        <Link href="/contact" :class="btnGlass">
                            <svg class="h-4 w-4" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7" /></svg>
                            <span>Get directions</span>
                        </Link>
                    </div>
                </div>
            </section>
        </div>

        <!-- ================= MODAL: QUICK VIEW ================= -->
        <Transition name="modal">
            <div
                v-if="quickView"
                class="fixed inset-0 z-50 flex items-end justify-center bg-[#06101e]/80 p-0 backdrop-blur-sm sm:items-center sm:p-4"
                @click.self="quickView = null"
                role="dialog"
                aria-modal="true"
                :aria-label="quickView.title"
            >
                <div class="modal-box relative max-h-[90dvh] w-full max-w-lg overflow-y-auto overscroll-contain rounded-t-3xl bg-white shadow-2xl sm:rounded-3xl">
                    <button
                        type="button"
                        @click="quickView = null"
                        aria-label="Close modal"
                        class="absolute right-3.5 top-3.5 z-10 flex h-9 w-9 items-center justify-center rounded-full bg-white/95 text-slate-700 shadow-md transition hover:scale-105 hover:bg-slate-100 cursor-pointer"
                    >
                        <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
                    </button>

                    <div class="relative aspect-[16/10] w-full overflow-hidden bg-slate-100">
                        <img :src="u(quickView.img, 1000)" :alt="quickView.title" class="vehicle-card-img h-full w-full object-cover" />
                        <span class="absolute bottom-3 left-3 rounded-md bg-black/70 px-2.5 py-1 text-xs font-semibold text-white backdrop-blur-sm">
                            {{ quickView.condition }} · {{ quickView.year }}
                        </span>
                        <span class="absolute left-3 top-3 rounded-md bg-amber-400 px-2.5 py-1 text-xs font-bold text-slate-950 shadow-md">
                            {{ quickView.badge }}
                        </span>
                    </div>

                    <div class="p-5 sm:p-6">
                        <p class="text-xs font-semibold uppercase tracking-wider text-slate-500">
                            Location: {{ quickView.location }}
                        </p>
                        <h3 class="mt-1 text-lg font-bold text-slate-900 sm:text-xl">{{ quickView.title }}</h3>
                        <p class="mt-1.5 text-2xl font-black text-[#0a1a30]">{{ formatPkr(quickView.price) }}</p>

                        <dl class="mt-4 grid grid-cols-2 gap-3 rounded-xl border border-slate-200 bg-slate-50 p-3.5 text-xs sm:text-sm">
                            <div>
                                <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Engine</dt>
                                <dd class="font-bold text-slate-800">{{ quickView.engine }}</dd>
                            </div>
                            <div>
                                <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Mileage</dt>
                                <dd class="font-bold text-slate-800">{{ quickView.mileage }}</dd>
                            </div>
                            <div>
                                <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Transmission</dt>
                                <dd class="font-bold text-slate-800">{{ quickView.transmission }}</dd>
                            </div>
                            <div>
                                <dt class="text-[11px] font-bold uppercase tracking-wider text-slate-400">Fuel Type</dt>
                                <dd class="font-bold text-slate-800">{{ quickView.fuel }}</dd>
                            </div>
                        </dl>

                        <div class="mt-5 flex flex-col gap-3 sm:flex-row">
                            <a
                                :href="waLink(`Assalam-o-Alaikum, I am interested in the ${quickView.title} priced at ${formatPkr(quickView.price)}.`)"
                                target="_blank"
                                rel="noopener"
                                :class="[btnGold, 'flex-1']"
                            >
                                <svg class="h-4 w-4 shrink-0" fill="currentColor" viewBox="0 0 24 24"><path d="M12.031 6.172c-3.181 0-5.767 2.586-5.768 5.766-.001 1.298.38 2.27 1.019 3.287l-.582 2.128 2.182-.573c.978.58 1.911.928 3.145.929 3.178 0 5.767-2.587 5.768-5.766.001-3.187-2.575-5.77-5.764-5.771zm3.392 8.244c-.144.405-.837.774-1.17.824-.299.045-.677.063-1.092-.069-.252-.08-.575-.187-.988-.365-1.739-.751-2.874-2.502-2.961-2.617-.087-.116-.708-.94-.708-1.793s.448-1.273.607-1.446c.159-.173.346-.217.462-.217l.332.007c.106.005.249-.04.39.298.144.347.491 1.2.534 1.287.043.087.072.188.014.304-.058.116-.087.188-.173.289l-.26.304c-.087.086-.177.18-.076.354.101.174.449.741.964 1.201.662.591 1.221.774 1.394.86.174.086.275.072.376-.043.101-.116.433-.506.549-.68.116-.173.231-.145.39-.087s1.011.477 1.184.564.289.13.332.202c.043.073.043.419-.101.824z" /></svg>
                                <span>Inquire on WhatsApp</span>
                            </a>
                            <button
                                type="button"
                                @click="quickView = null"
                                class="rounded-xl border-2 border-slate-200 px-6 py-3 text-sm font-bold text-slate-700 transition hover:border-[#0a1a30] hover:text-[#0a1a30] cursor-pointer"
                            >
                                Close
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </Transition>

        <!-- ================= MODAL: SELL REQUEST ================= -->
        <Transition name="modal">
            <div
                v-if="sellOpen"
                class="fixed inset-0 z-50 flex items-end justify-center bg-[#06101e]/80 p-0 backdrop-blur-sm sm:items-center sm:p-4"
                @click.self="sellOpen = false"
                role="dialog"
                aria-modal="true"
                aria-label="Sell your vehicle"
            >
                <div class="modal-box relative max-h-[92dvh] w-full max-w-md overflow-y-auto overscroll-contain rounded-t-3xl bg-white p-5 shadow-2xl sm:rounded-3xl sm:p-7">
                    <button
                        type="button"
                        @click="sellOpen = false"
                        aria-label="Close modal"
                        class="absolute right-3.5 top-3.5 flex h-9 w-9 items-center justify-center rounded-full bg-slate-100 text-slate-700 transition hover:scale-105 hover:bg-slate-200 cursor-pointer"
                    >
                        <svg class="h-5 w-5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
                    </button>

                    <h3 class="pr-8 text-xl font-black text-slate-900">Sell your vehicle</h3>
                    <p class="mt-1 text-xs sm:text-sm text-slate-600">Share your vehicle details and our valuation desk will call you with a market estimate.</p>

                    <form @submit.prevent="submitSell" class="mt-5 space-y-3.5 sm:space-y-4">
                        <div>
                            <label for="f-name" class="mb-1 block text-xs font-bold uppercase tracking-wider text-slate-700">Full Name</label>
                            <input id="f-name" v-model="sellForm.name" required type="text" autocomplete="name" placeholder="Muhammad Ali" :class="field" />
                        </div>
                        <div>
                            <label for="f-phone" class="mb-1 block text-xs font-bold uppercase tracking-wider text-slate-700">Phone or WhatsApp</label>
                            <input id="f-phone" v-model="sellForm.phone" required type="tel" autocomplete="tel" placeholder="0300 1234567" :class="field" />
                        </div>
                        <div class="grid gap-3 sm:grid-cols-2">
                            <div>
                                <label for="f-cat" class="mb-1 block text-xs font-bold uppercase tracking-wider text-slate-700">Category</label>
                                <select id="f-cat" v-model="sellForm.category" :class="field">
                                    <option value="cars">Car or SUV</option>
                                    <option value="bikes">Bike or Scooter</option>
                                    <option value="loaders">Loader or Commercial</option>
                                </select>
                            </div>
                            <div>
                                <label for="f-price" class="mb-1 block text-xs font-bold uppercase tracking-wider text-slate-700">Asking Price (PKR)</label>
                                <input id="f-price" v-model="sellForm.price" inputmode="numeric" type="text" placeholder="5,500,000" :class="field" />
                            </div>
                        </div>
                        <div>
                            <label for="f-veh" class="mb-1 block text-xs font-bold uppercase tracking-wider text-slate-700">Make, Model and Year</label>
                            <input id="f-veh" v-model="sellForm.vehicle" required type="text" placeholder="e.g. Toyota Corolla Altis 2022" :class="field" />
                        </div>
                        <button type="submit" :disabled="sent" :class="[btnGold, 'w-full py-3.5 mt-2']">
                            {{ sent ? "✓ Request Sent! We will call you soon." : "Submit Valuation Request" }}
                        </button>
                    </form>
                </div>
            </div>
        </Transition>
    </PublicSiteLayout>
</template>

<style>
/* ============================================================
   APEX SHOWROOM HOMEPAGE STYLES & RESPONSIVE RULES
   ============================================================ */

/* Hero Slide: Smooth cross-fade with gentle scale */
.hero-slide {
    height: 100% !important;
    width: 100% !important;
    max-width: none !important;
    object-fit: cover !important;
    opacity: 0;
    transform: scale(1.1);
    transition: opacity 1800ms ease-in-out, transform 8000ms ease-out;
    will-change: opacity, transform;
}
.hero-slide.is-on {
    opacity: 1;
    transform: scale(1);
}

/* Bulletproof image rules override for cards */
.cat-card img,
.vehicle-card-img {
    height: 100% !important;
    width: 100% !important;
    max-width: none !important;
    object-fit: cover !important;
}

/* Hero select element styling */
.hero-select {
    width: 100%;
    border-radius: 0.75rem;
    border: 1px solid rgba(255, 255, 255, 0.2);
    background-color: rgba(6, 16, 30, 0.85);
    padding: 0.75rem 1rem;
    font-size: 0.875rem;
    color: #ffffff;
    transition: border-color 0.2s ease, box-shadow 0.2s ease;
    cursor: pointer;
}
.hero-select:focus {
    outline: none;
    border-color: #fbbf24;
    box-shadow: 0 0 0 3px rgba(251, 191, 36, 0.25);
}
.hero-select option {
    background-color: #06101e;
    color: #ffffff;
}

/* Hero ambient glow */
.glow {
    animation: heroGlow 8s ease-in-out infinite;
}
@keyframes heroGlow {
    0%, 100% { opacity: 0.4; transform: translate(0, 0); }
    50% { opacity: 0.8; transform: translate(30px, -20px); }
}

/* Hero text orchestrated entrance */
.hero-in {
    opacity: 0;
    transform: translateY(20px);
    animation: heroIn 850ms cubic-bezier(0.2, 0.7, 0.2, 1) forwards;
    animation-delay: calc(var(--i) * 120ms + 100ms);
}
@keyframes heroIn {
    to {
        opacity: 1;
        transform: none;
    }
}

/* Gold shimmer headline */
.shimmer {
    background: linear-gradient(100deg, #fcd34d 20%, #ffffff 40%, #f59e0b 60%, #fcd34d 80%);
    background-size: 220% 100%;
    -webkit-background-clip: text;
    background-clip: text;
    color: transparent;
    animation: shimmer 5s linear infinite;
}
@keyframes shimmer {
    to {
        background-position: -220% 0;
    }
}

/* Brand marquee infinite loop */
.marquee {
    animation: marquee 35s linear infinite;
}
.marquee:hover {
    animation-play-state: paused;
}
@keyframes marquee {
    to {
        transform: translateX(-50%);
    }
}

/* Scroll reveal directive transition */
.rv {
    opacity: 0;
    transform: translateY(24px);
    transition: opacity 700ms ease, transform 700ms cubic-bezier(0.2, 0.7, 0.2, 1);
    transition-delay: var(--d, 0ms);
}
.rv.rv-in {
    opacity: 1;
    transform: none;
}

/* Sheen sweep on hover for vehicle cards */
.sheen {
    position: relative;
}
.sheen::after {
    content: "";
    position: absolute;
    inset: 0;
    background: linear-gradient(115deg, transparent 40%, rgba(255, 255, 255, 0.35) 50%, transparent 60%);
    transform: translateX(-120%);
    transition: transform 900ms ease;
    pointer-events: none;
}
.sheen:hover::after {
    transform: translateX(120%);
}

/* TransitionGroup for filtering vehicle cards */
.card-enter-active,
.card-leave-active {
    transition: opacity 300ms ease, transform 300ms ease;
}
.card-enter-from,
.card-leave-to {
    opacity: 0;
    transform: scale(0.96) translateY(8px);
}

/* Modal animations */
.modal-enter-active,
.modal-leave-active {
    transition: opacity 250ms ease;
}
.modal-enter-from,
.modal-leave-to {
    opacity: 0;
}
.modal-enter-active .modal-box {
    animation: pop 300ms cubic-bezier(0.2, 0.8, 0.2, 1);
}
@keyframes pop {
    from {
        opacity: 0;
        transform: translateY(30px) scale(0.97);
    }
}

/* Utility to hide scrollbars on horizontal lists */
.no-scrollbar::-webkit-scrollbar {
    display: none;
}
.no-scrollbar {
    -ms-overflow-style: none;
    scrollbar-width: none;
}

/* Accessibility: reduce motion */
@media (prefers-reduced-motion: reduce) {
    .hero-slide,
    .glow,
    .shimmer,
    .marquee,
    .hero-in,
    .rv,
    .sheen::after {
        animation: none !important;
        transition: none !important;
    }
    .hero-in,
    .rv {
        opacity: 1 !important;
        transform: none !important;
    }
}
</style>