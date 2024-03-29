<!-- Register Modal -->
<div class="modal fade" id="{dialogID}" tabindex="-1" aria-hidden="true">
  <div class="modal-dialog modal-sm modal-dialog-centered">
    <div class="modal-content">
      <div class="modal-header">
        <h4 class="modal-title">{$_("components.modals.register.title")}</h4>
        <button type="button" class="btn-close" data-bs-dismiss="modal"
        ></button>
      </div>
      <form on:submit|preventDefault="{onSubmit}">
        <div class="modal-body">
          <div class="mb-3">
            <SuccessAlert message="{$successMessage}" />
            <ErrorAlert error="{$error}" />
          </div>
          <div class="mb-3">
            <label for="registerUserName">{$_("components.modals.register.inputs.username")}</label>
            <input
              type="text"
              id="registerUserName"
              class="form-control"
              bind:value="{$data.username}" />
          </div>
          <div class="mb-3">
            <label for="registerEmail">{$_("components.modals.register.inputs.email")}</label>
            <input
              type="email"
              id="registerEmail"
              class="form-control"
              bind:value="{$data.email}" />
          </div>
          <div class="mb-3">
            <label for="registerPassword">{$_("components.modals.register.inputs.password")}</label>
            <input
              type="password"
              id="registerPassword"
              class="form-control"
              bind:value="{$data.password}" />
          </div>
          <div class="mb-3">
            <label for="registerPasswordRepeat">{$_("components.modals.register.inputs.password-repeat")}</label>
            <input
              type="password"
              id="registerPasswordRepeat"
              class="form-control"
              bind:value="{$data.passwordRepeat}" />
          </div>
          <div class="mb-3">
            <div class="form-check">
              <input
                type="checkbox"
                class="form-check-input"
                id="registerAcceptTerms"
                bind:checked="{$data.agreement}" />
              <label class="form-check-label" for="registerAcceptTerms">
                {@html $_("components.modals.register.inputs.agreement-text", {values: {link: `<a href="javascript:void(0);">${$_("components.modals.register.inputs.server-rules")}</a>`}})}
              </label>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button
            type="submit"
            class="btn btn-primary w-100"
            class:disabled="{loading}"
            disabled="{loading}">
            {$_("components.modals.register.register-button")}
          </button>
          <a
            href="javascript:void(0);"
            class="btn btn-link w-100"
            on:click="{() => {
              hide();
              showLoginModal();
            }}">
            {$_("components.modals.register.already-registered-button")}
          </a>
        </div>
      </form>
    </div>
  </div>
</div>

<script context="module">
  import { writable } from "svelte/store";

  const dialogID = "registerModal";
  const error = writable();
  const successMessage = writable();
  const dataDefault = {
    username: "",
    email: "",
    password: "",
    passwordRepeat: "",
    agreement: false,
    recaptcha: "",
  };
  const data = writable({ ...dataDefault });

  let callback = () => {};
  let hideCallback = () => {};
  let modal;

  export function show() {
    error.set(null);
    data.set({ ...dataDefault });

    modal = new window.bootstrap.Modal(document.getElementById(dialogID), {
      backdrop: "static",
      keyboard: false,
    });
    modal.show();
  }

  export function hide() {
    hideCallback();

    modal.hide();
  }

  export function setCallback(newCallback) {
    callback = newCallback;
  }

  export function onHide(newCallback) {
    hideCallback = newCallback;
  }
</script>

<script>
  import { _ } from "svelte-i18n";
  import { get } from "svelte/store";

  import { NETWORK_ERROR } from "$lib/api.util";

  import { show as showLoginModal } from "./LoginModal.svelte";

  import ErrorAlert from "$lib/component/ErrorAlert.svelte";
  import SuccessAlert from "$lib/component/SuccessAlert.svelte";

  import { sendRegister } from "$lib/services/auth.js";

  let loading = false;

  async function onSubmit() {
    error.set(null);
    successMessage.set(null);

    loading = true;

    await sendRegister(get(data))
      .then((body) => {
        loading = false;

        if (body.result === "ok") {
          successMessage.set("REGISTER_SUCCESSFUL");

          callback(get(data));

          data.set({ ...dataDefault });
        } else {
          error.set(body.result === "error" ? body.error : NETWORK_ERROR);
        }
      })
      .catch(() => {
        loading = false;

        error.set(NETWORK_ERROR);
      });
  }
</script>
