<script setup>
import { Link, usePage } from "@inertiajs/vue3";
import { computed, onMounted, onUnmounted, ref, watch } from "vue";
import InstitutionLogo from "@/Components/PublicSite/InstitutionLogo.vue";
import PublicSiteNavLink from "@/Components/PublicSite/PublicSiteNavLink.vue";

const page = usePage();
const cms = computed(() => page.props.institution || {});
const name = computed(
    () => cms.value.name || "Bright Future Educational Institute",
);
const studentPortalUrl = computed(
    () => cms.value.student_portal_url || "/login",
);
const lmsUrl = computed(() => cms.value.lms_url || "/login");

const isMobileMenuOpen = ref(false);
const isDesktop = ref(
    typeof window !== "undefined" ? window.innerWidth >= 1024 : false,
);

const navItems = [
    { label: "Home", href: route("publicSite.home"), routeName: "publicSite.home" },
    { label: "About", href: route("publicSite.about"), routeName: "publicSite.about" },
    {
        label: "Academics",
        href: route("publicSite.academics"),
        routeName: "publicSite.academics",
    },
    {
        label: "Admissions",
        href: route("publicSite.admissions"),
        routeName: "publicSite.admissions",
    },
    {
        label: "Campus Life",
        href: route("publicSite.campus"),
        routeName: "publicSite.campus",
    },
    {
        label: "News & Events",
        href: route("publicSite.news"),
        routeName: "publicSite.news",
    },
    {
        label: "Contact",
        href: route("publicSite.contact"),
        routeName: "publicSite.contact",
    },
];

const isActive = (item) => route().current(item.routeName);

const closeMobileMenu = () => {
    isMobileMenuOpen.value = false;
};

let mediaQuery = null;

const syncViewport = () => {
    isDesktop.value = mediaQuery ? mediaQuery.matches : false;
    if (isDesktop.value) {
        closeMobileMenu();
    }
};

watch(
    () => page.url,
    () => closeMobileMenu(),
);

watch(isMobileMenuOpen, (open) => {
    document.body.style.overflow = open ? "hidden" : "";
});

onMounted(() => {
    mediaQuery = window.matchMedia("(min-width: 1024px)");
    syncViewport();
    mediaQuery.addEventListener("change", syncViewport);
});

onUnmounted(() => {
    if (mediaQuery) {
        mediaQuery.removeEventListener("change", syncViewport);
    }
    document.body.style.removeProperty("overflow");
});
</script>

<template>
    <div class="ps-chrome">
        <header class="ps-header">
            <nav class="ps-container ps-header-inner" aria-label="Primary">
                <Link
                    :href="route('publicSite.home')"
                    class="ps-brand"
                    :aria-label="name"
                    @click="closeMobileMenu"
                >
                    <span class="ps-brand-mark">
                        <InstitutionLogo />
                    </span>
                </Link>

                <div v-if="isDesktop" class="ps-nav">
                    <PublicSiteNavLink
                        v-for="item in navItems"
                        :key="item.routeName"
                        :href="item.href"
                        :active="isActive(item)"
                    >
                        {{ item.label }}
                    </PublicSiteNavLink>
                </div>

                <div class="ps-header-actions">
                    <a
                        v-if="isDesktop"
                        :href="studentPortalUrl"
                        class="ps-util-link"
                    >
                        Student Portal
                    </a>
                    <a v-if="isDesktop" :href="lmsUrl" class="ps-util-link">
                        LMS
                    </a>
                    <Link
                        :href="route('publicSite.admissions')"
                        class="ps-btn ps-btn-primary"
                    >
                        Apply Now
                    </Link>
                    <button
                        v-if="!isDesktop"
                        type="button"
                        class="ps-menu-btn"
                        :aria-expanded="isMobileMenuOpen"
                        aria-controls="public-mobile-menu"
                        :aria-label="
                            isMobileMenuOpen ? 'Close menu' : 'Open menu'
                        "
                        @click="isMobileMenuOpen = !isMobileMenuOpen"
                    >
                        <svg
                            class="h-5 w-5"
                            fill="none"
                            stroke="currentColor"
                            viewBox="0 0 24 24"
                        >
                            <path
                                stroke-linecap="round"
                                stroke-linejoin="round"
                                stroke-width="2"
                                :d="
                                    isMobileMenuOpen
                                        ? 'M6 18L18 6M6 6l12 12'
                                        : 'M4 6h16M4 12h16M4 18h16'
                                "
                            />
                        </svg>
                    </button>
                </div>
            </nav>

            <div
                v-if="!isDesktop && isMobileMenuOpen"
                id="public-mobile-menu"
                class="ps-mobile-panel"
            >
                <div
                    class="ps-container"
                    style="padding-top: 0.75rem; padding-bottom: 1.25rem"
                >
                    <PublicSiteNavLink
                        v-for="item in navItems"
                        :key="`m-${item.routeName}`"
                        :href="item.href"
                        :active="isActive(item)"
                        mobile
                        @navigate="closeMobileMenu"
                    >
                        {{ item.label }}
                    </PublicSiteNavLink>
                    <a
                        :href="studentPortalUrl"
                        class="ps-mobile-link"
                        @click="closeMobileMenu"
                    >
                        Student Portal
                    </a>
                    <a
                        :href="lmsUrl"
                        class="ps-mobile-link"
                        @click="closeMobileMenu"
                    >
                        LMS
                    </a>
                </div>
            </div>
        </header>
    </div>

    <div
        v-if="!isDesktop && isMobileMenuOpen"
        class="ps-overlay"
        aria-hidden="true"
        @click="closeMobileMenu"
    />
</template>

<style>
/* ============================================================
   Home page colour theme (navy + amber gold)
   navy   #06101e / #0a1a30
   gold   #fbbf24 (amber-400) / #f59e0b (amber-500) / #fcd34d (amber-300)
   text   #e2e8f0 (slate-200) / #cbd5e1 (slate-300)
   Colours only. Layout and spacing are not changed.
   ============================================================ */

/* ---------- Header bar ---------- */
.ps-chrome .ps-header {
    background: rgba(6, 16, 30, 0.92);
    -webkit-backdrop-filter: blur(14px);
    backdrop-filter: blur(14px);
    border-bottom: 1px solid rgba(251, 191, 36, 0.18);
    box-shadow: 0 8px 30px rgba(6, 16, 30, 0.35);
    color: #e2e8f0;
}

/* ---------- Brand ---------- */
.ps-chrome .ps-brand,
.ps-chrome .ps-brand-mark {
    color: #ffffff;
}

/* ---------- Desktop nav links ---------- */
.ps-chrome .ps-nav a {
    color: #cbd5e1;
    transition: color 0.25s ease, background-color 0.25s ease, border-color 0.25s ease;
}
.ps-chrome .ps-nav a:hover,
.ps-chrome .ps-nav a:focus-visible {
    color: #fcd34d;
    background-color: rgba(251, 191, 36, 0.1);
}
/* active link (works with common active patterns) */
.ps-chrome .ps-nav a[aria-current="page"],
.ps-chrome .ps-nav a.active,
.ps-chrome .ps-nav a.is-active,
.ps-chrome .ps-nav a.ps-nav-link-active {
    color: #fbbf24;
    background-color: rgba(251, 191, 36, 0.12);
    border-color: #fbbf24;
}

/* ---------- Student Portal / LMS ---------- */
.ps-chrome .ps-util-link {
    color: #cbd5e1;
    transition: color 0.25s ease;
}
.ps-chrome .ps-util-link:hover,
.ps-chrome .ps-util-link:focus-visible {
    color: #fcd34d;
}

/* ---------- Apply Now button (same as home gold button) ---------- */
.ps-chrome .ps-btn-primary {
    background-image: linear-gradient(to right, #fbbf24, #f59e0b);
    background-color: #f59e0b;
    color: #020617;
    border: 0;
    font-weight: 700;
    box-shadow: 0 10px 24px rgba(245, 158, 11, 0.25);
    transition: transform 0.3s ease, box-shadow 0.3s ease;
}
.ps-chrome .ps-btn-primary:hover,
.ps-chrome .ps-btn-primary:focus-visible {
    color: #020617;
    transform: translateY(-2px);
    box-shadow: 0 14px 30px rgba(245, 158, 11, 0.4);
}

/* ---------- Mobile menu button ---------- */
.ps-chrome .ps-menu-btn {
    color: #fcd34d;
    background-color: rgba(255, 255, 255, 0.08);
    border: 1px solid rgba(255, 255, 255, 0.2);
    transition: background-color 0.25s ease, color 0.25s ease;
}
.ps-chrome .ps-menu-btn:hover,
.ps-chrome .ps-menu-btn:focus-visible {
    background-color: #fbbf24;
    color: #020617;
}

/* ---------- Mobile panel ---------- */
.ps-chrome .ps-mobile-panel {
    background: #0a1a30;
    border-top: 1px solid rgba(251, 191, 36, 0.2);
    box-shadow: 0 20px 40px rgba(6, 16, 30, 0.5);
    max-height: calc(100svh - 5.25rem);
    overflow-y: auto;
    -webkit-overflow-scrolling: touch;
    overscroll-behavior: contain;
}
.ps-chrome .ps-mobile-panel a {
    color: #e2e8f0;
    border-color: rgba(255, 255, 255, 0.08);
    transition: color 0.25s ease, background-color 0.25s ease;
}
.ps-chrome .ps-mobile-panel a:hover,
.ps-chrome .ps-mobile-panel a:focus-visible {
    color: #fcd34d;
    background-color: rgba(251, 191, 36, 0.1);
}
.ps-chrome .ps-mobile-panel a[aria-current="page"],
.ps-chrome .ps-mobile-panel a.active,
.ps-chrome .ps-mobile-panel a.is-active {
    color: #fbbf24;
    background-color: rgba(251, 191, 36, 0.12);
}

/* ---------- Overlay behind mobile menu ---------- */
.ps-overlay {
    background: rgba(6, 16, 30, 0.7);
    -webkit-backdrop-filter: blur(4px);
    backdrop-filter: blur(4px);
}
</style>