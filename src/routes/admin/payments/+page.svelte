<script>
    import Navbar from "../../../components/Navbar.svelte";
    import Sidebar from "../../../components/Sidebar.svelte";
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import Payment from "../../../components/Payment.svelte";

    let sidebarOpen = false;

    
  onMount(() => {
    const token = localStorage.getItem('token');
    if (!token) {
      goto('/admin', { replaceState: true });
    }
  });
</script>

<div class="min-h-screen lg:grid lg:grid-cols-[256px_1fr]">

    <aside class="fixed inset-y-0 left-0 z-50 w-64 bg-base-200 transform transition-transform lg:relative lg:translate-x-0 {sidebarOpen ? 'translate-x-0' : '-translate-x-full'}">
        <Sidebar />
    </aside>
    
    {#if sidebarOpen}
        <div class="fixed inset-0 bg-black bg-opacity-50 z-40 lg:hidden" on:click={() => sidebarOpen = false}></div>
    {/if}
    
    <div class="flex flex-col min-h-screen">
        <header class="sticky top-0 z-30 bg-base-100 shadow-sm m-3 rounded-lg">
            <div class="flex items-center gap-4">
                <button class="lg:hidden ms-2" on:click={() => sidebarOpen = !sidebarOpen}>
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"/>
                      </svg> 
                </button>
                <Navbar />
            </div>
        </header>
        
        <main class="flex-1 p-4 lg:p-6">
            <Payment/>
        </main>
    </div>
</div>