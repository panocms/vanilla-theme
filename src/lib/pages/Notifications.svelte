<div class="container">
  <!-- Action Menu -->
  <div
    class="row justify-content-end mb-3 animate__animated animate__slideInUp">
    {#if $notifications.length !== 0}
      <div class="col-auto">
        <button
          type="button"
          class="btn btn-danger"
          on:click="{() => onDeleteAllClick()}"
          >Tümünü Sil
        </button>
      </div>
    {/if}
  </div>

  <!-- All Notifications -->

  <div class="card">
    <div class="card-body">
      {#each $notifications as notification, index (notification)}
        <div class="list-group w-100 flex-row align-items-center">
          <a
            href="javascript:void(0);"
            class="list-group-item list-group-item-action  d-flex flex-row w-100"
            class:notification-unread="{notification.status === 'NOT_READ'}">
            <div class="col-auto">
              <i class="fa fa-bell mx-3 my-3 text-primary"></i>
            </div>
            <div class="col">
              <span class="text-wrap text-dark">{notification.typeId}</span>
              <small class="text-gray d-block">
                {getTime(checkTime, parseInt(notification.date), "")}
              </small>
            </div>
          </a>
          <button
            class="btn-close text-danger btn-sm mx-2"
            use:tooltip="{['Bildirimi Sil', { placement: 'right' }]}"
            on:click="{deleteNotification(notification.id)}">
          </button>
        </div>
      {/each}

      {#if $notifications.length === 0}
        <div
          class="d-flex flex-column align-items-center justify-content-center">
          <i class="fas fa-bell fa-3x text-dark text-opacity-25 m-3"></i>
          <p class="text-gray">Burada içerik yok.</p>
        </div>
      {/if}

      {#if $notifications.length < $count && $count > 10 + 10 * page}
        <div class="mt-3">
          <button
            class="btn btn-link bg-light d-block m-auto"
            class:disabled="{loadMoreLoading}"
            on:click="{loadMore}"
            >Daha Fazla Göster ({$count - $notifications.length})
          </button>
        </div>
      {/if}
    </div>
  </div>
</div>

<ConfirmRemoveAllNotificationsModal />

<script context="module">
  import { writable, get } from "svelte/store";

  import { browser } from "$app/environment";

  import ApiUtil from "$lib/api.util.js";

  const notifications = writable([]);
  const count = writable(0);

  Array.prototype.insert = function (index, item) {
    this.splice(index, 0, item);

    return this;
  };

  Array.prototype.remove = function (index) {
    this.splice(index, 1);

    return this;
  };

  function setNotifications(newNotifications) {
    if (get(notifications).length === 0 || newNotifications.length === 0)
      notifications.set(newNotifications);
    else {
      const listOfFilterIsNotificationExists = [];

      newNotifications.forEach((item, index) => {
        listOfFilterIsNotificationExists[index] = get(notifications).filter(
          (filterItem) => filterItem.id === item.id
        );
      });

      newNotifications.forEach((item, index) => {
        if (listOfFilterIsNotificationExists[index].length === 0) {
          notifications.set(get(notifications).insert(index, item));
        }
      });
    }
  }

  async function loadData({ request, CSRFToken }) {
    return new Promise((resolve, reject) => {
      ApiUtil.get({
        path: "/api/notifications",
        request,
        CSRFToken,
      }).then((body) => {
        if (body.result === "ok") {
          resolve(body);
        } else {
          reject(body);
        }
      });
    });
  }

  /**
   * @type {import('@sveltejs/kit').PageLoad}
   */
  export async function load(event) {
    const { parent } = event;
    await parent();

    let data = {};

    // if (event.stuff.NETWORK_ERROR) {
    //   output.props.data.NETWORK_ERROR = true;
    //
    //   return output;
    // }

    await loadData({ request: event }).then((body) => {
      setNotifications(body.notifications);

      count.set(parseInt(body.notificationCount));
    });

    return data;
  }
</script>

<script>
  import { onDestroy, onMount } from "svelte";
  import { formatDistanceToNow } from "date-fns";

  import tooltip from "$lib/tooltip.util.js";
  import { session } from "$lib/Store.js";

  import ConfirmRemoveAllNotificationsModal, {
    show as showDeleteAllNotificationsModal,
    setCallback as setDeleteAllNotificationsModalCallback,
  } from "$lib/component/modals/ConfirmRemoveAllNotificationsModal.svelte";

  export let data;

  let notificationProcessID = 0;
  let page = 0;
  let loadMoreLoading = false;

  let checkTime = 0;
  let interval;

  function getNotifications(id) {
    loadData({ CSRFToken: $session.CSRFToken }).then((data) => {
      if (notificationProcessID === id) {
        if (data.result === "ok") {
          setNotifications(data.notifications);

          count.set(parseInt(data.notificationCount));
        }

        setTimeout(() => {
          if (notificationProcessID === id) {
            startNotificationsCountdown();
          }
        }, 1000);
      }
    });
  }

  function loadMore() {
    loadMoreLoading = true;

    ApiUtil.get({
      path: `/api/notifications/${
        get(notifications)[get(notifications).length - 1].id
      }/more`,
      CSRFToken: $session.CSRFToken,
    }).then((body) => {
      if (body.result === "ok") {
        body.notifications.forEach((notification) => {
          notifications.update((value) =>
            value.insert(value.length, notification)
          );
        });

        loadMoreLoading = false;
      }
    });
  }

  function deleteNotification(id) {
    ApiUtil.delete({
      path: `/api/notifications/${id}`,
      CSRFToken: $session.CSRFToken,
    }).then((body) => {
      if (body.result === "ok") {
        get(notifications).forEach((notification) => {
          if (notification.id === id) {
            notifications.update((value) =>
              value.remove(value.indexOf(notification))
            );

            count.update((value) => value--);
          }
        });
      }
    });
  }

  function startNotificationsCountdown() {
    notificationProcessID++;

    const id = notificationProcessID;

    getNotifications(id);
  }

  function stopNotificationsCountdown() {
    notificationProcessID++;

    clearInterval(interval);
  }

  function getTime(check, time, locale) {
    return formatDistanceToNow(time, { addSuffix: true });
  }

  function onDeleteAllClick() {
    stopNotificationsCountdown();

    showDeleteAllNotificationsModal();
  }

  if (browser) startNotificationsCountdown();

  onMount(() => {
    interval = setInterval(() => {
      checkTime += 1;
    }, 1000);
  });

  onDestroy(() => {
    stopNotificationsCountdown();
  });

  setDeleteAllNotificationsModalCallback(() => {
    startNotificationsCountdown();
  });
</script>