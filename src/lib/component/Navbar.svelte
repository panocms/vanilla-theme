<!-- Navbar -->
<div class="container">
  <nav class="navbar navbar-expand-lg navbar-dark bg-primary rounded shadow-lg">
    <button
      aria-controls="navbar"
      aria-expanded="false"
      aria-label="Toggle navigation"
      class="navbar-toggler d-lg-none"
      data-target="#navbar"
      data-toggle="collapse"
      type="button">
      <i aria-hidden="true" class="fa fa-bars"></i>
    </button>

    <ul class="navbar-nav flex-row ml-auto order-lg-last">
      {#if $session.user}
        <li class="nav-item mr-lg-0 mr-5">
          <div class="dropdown">
            <a
              class="nav-link"
              href="javascript:void(0);"
              id="playerMenuDropdown"
              data-toggle="dropdown"
              aria-haspopup="true"
              aria-expanded="false">
              <i class="fas fa-user mr-2" aria-hidden="true"></i>
              {$session.user.username}
            </a>
            <div class="dropdown-menu" aria-labelledby="playerMenuDropdown">
              <a class="dropdown-item" href="/profile">Profil</a>
              <a class="dropdown-item" href="/tickets">Talepler</a>
              <a class="dropdown-item" href="/profile/settings">Ayarlar</a>
              <a
                class="dropdown-item text-danger"
                href="javascript:void(0);"
                on:click="{logout}">Çıkış Yap</a>
            </div>
          </div>
        </li>
      {:else}
        <li class="nav-item mr-lg-0 mr-5">
          <a
            class="nav-link"
            href="javascript:void(0);"
            on:click="{showLoginModal}">
            GİRİŞ YAP
          </a>
        </li>
        <li class="nav-item">
          <a
            class="nav-link"
            href="javascript:void(0);"
            on:click="{showRegisterModal}">
            KAYIT OL
          </a>
        </li>
      {/if}
    </ul>

    <div class="collapse navbar-collapse" id="navbar">
      <ul class="navbar-nav text-lg-left text-center mr-auto mt-2 mt-lg-0">
        <li class="nav-item active">
          <a class="nav-link" href="/">ANA SAYFA</a>
        </li>
      </ul>
    </div>
  </nav>
</div>

<!-- Navbar -->
<script>
  import { session } from "$app/stores";

  import ApiUtil from "$lib/api.util";

  import { show as showLoginModal } from "./modals/LoginModal.svelte";
  import { show as showRegisterModal } from "./modals/RegisterModal.svelte";

  async function logout() {
    await ApiUtil.post({path: "/auth/logout", CSRFToken: $session.CSRFToken}).then(() => {
      $session.user = null;
      $session.CSRFToken = null;
    });
  }
</script>