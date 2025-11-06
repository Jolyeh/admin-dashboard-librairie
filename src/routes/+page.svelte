<script>
    import { onMount } from "svelte";
    import { urlApi } from "../utils/constant.js";
    import { Toast } from "../utils/toast.js";

    let books = [];
    let users = [];
    let totalBooks = 0;
    let totalReader = 0;
    let totalAuthors = 0;

    async function fetchBooks() {
        try {
            const response = await fetch(`${urlApi}/book`, {
                headers: { "Content-Type": "application/json" },
            });
            const resp = await response.json();

            if (resp.status) {
                books = resp.data;
                totalBooks = books.length;
            } else {
                Toast.error(resp.message || "Échec du chargement des livres.");
            }
        } catch (error) {
            console.error("Erreur lors du chargement des livres :", error);
            Toast.error("Impossible de charger les livres.");
        }
    }

    async function fetchUsers() {
        try {
            const response = await fetch(`${urlApi}/user`, {
                headers: { "Content-Type": "application/json" },
            });
            const resp = await response.json();

            if (resp.status) {
                users = resp.data;
                totalReader = users.filter(u => u.role?.name === "BUYER").length;
                totalAuthors = users.filter(u => u.role?.name === "SELLER").length;
            } else {
                Toast.error(resp.message || "Échec du chargement des utilisateurs.");
            }
        } catch (error) {
            console.error("Erreur lors du chargement des utilisateurs :", error);
            Toast.error("Impossible de charger les utilisateurs.");
        }
    }

    onMount(async () => {
        await Promise.all([fetchBooks(), fetchUsers()]);
    });
</script>

<section class="hero min-h-screen bg-gradient-to-br from-primary/10 via-base-100 to-secondary/10">
    <div class="hero-content flex flex-col items-center justify-center text-center px-6 py-10 md:py-20">
        <!-- Icône principale -->
        <div class="mb-8 animate-bounce">
            <img src="images/icon.png" class="h-25 w-25" alt="icon" srcset="">
        </div>

        <!-- Titre principal -->
        <h1 class="text-3xl md:text-5xl font-extrabold text-primary mb-4">
            Bienvenue sur J-Librairie
        </h1>

        <!-- Sous-titre -->
        <p class="text-lg md:text-xl text-base-content/80 font-medium mb-6">
            Votre librairie numérique africaine
        </p>

        <!-- Description -->
        <p class="text-base md:text-lg text-base-content/70 max-w-2xl mx-auto mb-10 leading-relaxed">
            Découvrez des milliers de livres, magazines et contenus éducatifs en français et en langues locales.
            Achetez, téléchargez et lisez hors ligne sur notre application mobile conviviale.
        </p>

        <!-- Section des statistiques -->
        <div class="stats shadow w-full max-w-4xl mx-auto flex flex-col sm:flex-row">
            <!-- Livres -->
            <div class="stat">
                <div class="stat-title">Livres</div>
                <div class="stat-value text-primary">{totalBooks}</div>
                <div class="stat-desc">Disponibles sur la plateforme</div>
            </div>

            <!-- Lecteurs -->
            <div class="stat">
                <div class="stat-title">Lecteurs</div>
                <div class="stat-value text-primary">{totalReader}</div>
                <div class="stat-desc">Inscrits et actifs</div>
            </div>

            <!-- Auteurs / Vendeurs -->
            <div class="stat">
                <div class="stat-title">Auteurs / Vendeurs</div>
                <div class="stat-value text-primary">{totalAuthors}</div>
                <div class="stat-desc">Contributeurs actifs</div>
            </div>
        </div>
    </div>
</section>
