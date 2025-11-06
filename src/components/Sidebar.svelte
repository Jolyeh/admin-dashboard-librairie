<script>
    import { page } from "$app/stores";

    const menuItems = [
        { label: "Tableau de bord", href: "/admin/dashboard" },
        { label: "Utilisateurs", href: "/admin/users" },
        { label: "Livres", href: "/admin/books" },
        { label: "Catégories", href: "/admin/categories" },
        { label: "Paiements", href: "/admin/payments" },
        { label: "Demandes", href: "/admin/requests" },
    ];

    // Fonction réactive pour vérifier si le lien est actif
    $: isActive = (href) => $page.url.pathname === href;
</script>

<div class="drawer lg:drawer-open">
    <input id="my-drawer-3" type="checkbox" class="drawer-toggle" />

    <ul class="menu bg-base-200 min-h-full p-4">
        <li>
            <a
                href="/"
                class="mb-5 mt-3 text-xl font-bold hover:none focus:outline-none focus:ring-0 hover:bg-transparent hover:text-inherit"
            >
                J-Librairie
            </a>
        </li>

        {#each menuItems as item}
            <li>
                <a
                    href={item.href}
                    class="font-medium mb-3 text-base focus:bg-primary focus:text-white"
                    class:bg-primary={isActive(item.href)}
                    class:text-white={isActive(item.href)}
                >
                    {item.label}
                </a>
            </li>
        {/each}
    </ul>

    <!-- Sidebar -->
    <div class="drawer-side">
        <label
            for="my-drawer-3"
            aria-label="close sidebar"
            class="drawer-overlay"
        ></label>
        <ul class="menu bg-base-200 min-h-full w-65 p-4">
            <li>
                <a
                    href="/"
                    class="mb-5 mt-3 text-xl font-bold hover:none focus:outline-none focus:ring-0 hover:bg-transparent hover:text-inherit"
                >
                    J-Librairie
                </a>
            </li>

            {#each menuItems as item}
                <li>
                    <a
                        href={item.href}
                        class="font-medium mb-3 text-base focus:bg-primary focus:text-white"
                        class:bg-primary={isActive(item.href)}
                        class:text-white={isActive(item.href)}
                    >
                        {item.label}
                    </a>
                </li>
            {/each}
        </ul>
    </div>
</div>
