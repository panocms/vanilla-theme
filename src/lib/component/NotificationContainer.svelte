<div class="toast-container position-fixed bottom-0 end-0 p-3 d-xl-block d-none">
  {#each $notifications as notification, index (notification)}
    <div
      id="notificationToast{notification.id}"
      class="toast"
      role="alert"
      aria-live="assertive"
      aria-atomic="true"
      on:click="{() => onClick(notification)}">
      <div class="toast-header text-bg-primary">
        <strong class="me-auto">Pano</strong>
        <small>{getTime(checkTime, parseInt(notification.date), locales[$currentLanguage['date-fns-code']])}</small>
        <button
          type="button"
          class="btn-close btn-close-white"
          data-bs-dismiss="toast"></button>
      </div>
      <div class="toast-body">{notification.type}</div>
    </div>
  {/each}
</div>

<script context="module">
  import { tick } from "svelte";
  import { get, writable } from "svelte/store";

  const notifications = writable([]);
  const notificationToasts = writable({});

  Array.prototype.insert = function (index, item) {
    this.splice(index, 0, item);

    return this;
  };

  Array.prototype.remove = function (index) {
    this.splice(index, 1);

    return this;
  };

  function delay(time) {
    return new Promise((resolve) => setTimeout(resolve, time));
  }

  function deleteFromNotifications(id) {
    notifications.update((notifications) => {
      const foundNotification = notifications.find(
        (notification) => notification.id === id
      );

      notifications.remove(notifications.indexOf(foundNotification));
      delete notificationToasts[id];

      return notifications;
    });
  }

  export async function show(id) {
    while (!window.bootstrap) {
      await delay(50);
    }

    await tick();

    const notificationElement = document.getElementById(
      "notificationToast" + id
    );

    if (notificationElement) {
      const toast = new window.bootstrap.Toast(notificationElement);

      notificationToasts[id] = toast;

      toast.show();

      notificationElement.addEventListener("hidden.bs.toast", () => {
        deleteFromNotifications(id);
      });
    }
  }

  export async function hide(id) {
    notificationToasts[id].hide();
    deleteFromNotifications(id);
  }
</script>

<script>
  import { getContext, onDestroy, onMount } from "svelte";
  import { formatDistanceToNow } from "date-fns";

  import { browser } from "$app/environment";

  import { notificationsCount, quickNotifications } from "$lib/Store";
  import ApiUtil from "$lib/api.util";
  import { onNotificationClick } from "$lib/NotificationManager.js";
  import * as locales from "date-fns/locale";
  import { currentLanguage } from "$lib/language.util.js";

  let quickNotificationProcessID = 0;

  let checkTime = 0;
  let interval;

  const session = getContext("session");

  function getTime(check, time, locale) {
    return formatDistanceToNow(time, { addSuffix: true, locale });
  }

  function addNotification(notification) {
    notifications.update((notifications) => {
      notifications.push(notification);

      return notifications;
    });

    show(notification.id);
  }

  function setNotifications(newNotifications) {
    if (get(quickNotifications).length === 0 || newNotifications.length === 0) {
      quickNotifications.set(newNotifications);

      newNotifications.forEach((notification) => {
        if (notification.status === "NOT_READ") {
          addNotification(notification);
        }
      });

      return;
    }

    const listOfFilterIsNotificationExists = [];

    newNotifications.forEach((item, index) => {
      listOfFilterIsNotificationExists[index] = get(quickNotifications).filter(
        (filterItem) => filterItem.id === item.id
      );
    });

    newNotifications.forEach((item, index) => {
      if (listOfFilterIsNotificationExists[index].length === 0) {
        quickNotifications.update((quickNotifications) => {
          return quickNotifications.insert(index, item);
        });

        addNotification(item);
      }
    });

    get(quickNotifications).forEach((item, index) => {
      const newArrayOfFilter = newNotifications.filter(
        (filterItem) => filterItem.id === item.id
      );

      if (newArrayOfFilter.length === 0) {
        quickNotifications.update((quickNotifications) => {
          return quickNotifications.remove(index);
        });
      }
    });
  }

  function getQuickNotifications(id) {
    ApiUtil.get({
      path: "/api/notifications/quick",
    }).then((body) => {
      if (quickNotificationProcessID === id) {
        if (body.result === "ok") {
          setNotifications(body.notifications);

          notificationsCount.set(body.notificationCount);
        }

        setTimeout(() => {
          if (quickNotificationProcessID === id) {
            startQuickNotificationsCountDown();
          }
        }, 1000);
      }
    });
  }

  function startQuickNotificationsCountDown() {
    quickNotificationProcessID++;

    const id = quickNotificationProcessID;

    getQuickNotifications(id);
  }

  if (browser) {
    const sessionSubscription = session.subscribe((session) => {
      quickNotificationProcessID++;

      if (session.user) {
        startQuickNotificationsCountDown();
      }
    });

    onDestroy(sessionSubscription);
  }

  function markRead(id) {
    ApiUtil.post({
      path: `/api/notifications/${id}/read`,
    });
  }

  function onClick(notification) {
    markRead(notification.id);
    onNotificationClick(notification);
    hide(notification.id);
  }

  onMount(() => {
    interval = setInterval(() => {
      checkTime += 1;
    }, 1000);
  });

  onDestroy(() => {
    quickNotificationProcessID++;
    clearInterval(interval);
  });
</script>
