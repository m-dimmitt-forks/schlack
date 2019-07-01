# Singleton-Driven UI - Notification Center

While components are generally reusable, sometimes it's more convenient to use _singleton state_ to drive a component, particularly in cases where many parts of a codebase need to be able to get involved with its behavior. A great example of this kind of pattern is a notification center.

## The Notification Component

Let's begin with a `<Notification />` component. Create the file [`app/templates/components/notification.hbs`](../app/templates/components/notification.hbs) as follows

```hbs
<div class="notification flex flex-row items-center bg-{{@notification.color}} text-white text-sm font-bold px-4 py-3 notification-transition {{if @notification.entering "entering" ""}} {{if @notification.leaving "leaving" ""}}"
    role="alert">
  {{#if hasBlock}}
    {{yield }}
  {{else}}
    <svg class="fill-current w-4 h-4 mr-2" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 20">
      <path
        d="M12.432 0c1.34 0 2.01.912 2.01 1.957 0 1.305-1.164 2.512-2.679 2.512-1.269 0-2.009-.75-1.974-1.99C9.789 1.436 10.67 0 12.432 0zM8.309 20c-1.058 0-1.833-.652-1.093-3.524l1.214-5.092c.211-.814.246-1.141 0-1.141-.317 0-1.689.562-2.502 1.117l-.528-.88c2.572-2.186 5.531-3.467 6.801-3.467 1.057 0 1.233 1.273.705 3.23l-1.391 5.352c-.246.945-.141 1.271.106 1.271.317 0 1.357-.392 2.379-1.207l.6.814C12.098 19.02 9.365 20 8.309 20z" />
    </svg>
    <p>{{@notification.body}}</p>
  {{/if}}
</div>
```

This component uses a pattern where we can have a default DOM structure for notifications, in the event we use the "inline form" of the component

```hbs
<Notification @notification=... />
```

but can also customize it by using the "block form"

```hbs
<Notification @notification=... >
  Something custom
</Notification>
```

In the component's `.hbs` file above, you can see a condition based on the `hasBlock` property -- this is what allows us to conditionally render the SVG _only_ when inline form is used.

## The Service

Next, let's build a service to hold the collection of notifications

```sh
ember generate service notifications
```

and implement [`app/services/notifications.js`](../app/services/notifications.js) as follows

```js
import Service from '@ember/service';
import { action, set as emberSet } from '@ember/object';
import { tracked } from '@glimmer/tracking';

const getId = () => Math.round(Math.random() * 10000000);

const HANG_TIME = 3000; // ms

export default class NotificationsService extends Service {
  @tracked
  messages = [];

  @action
  notify(body, color = 'indigo-darker') {
    const id = getId();
    const newNotification = {
      id,
      body,
      color,
      entering: false,
      leaving: false,
    };
    // adds a css class to animate in
    emberSet(newNotification, 'entering', true);

    // tracked property update via assignment
    this.messages = [...this.messages, newNotification];

    // REMOVE after elapsed time is complete
    setTimeout(() => {
      // adds a css class to animate out
      emberSet(newNotification, 'leaving', true);

      // remove notification by ID
      const idx = this.messages.map(n => `${n.id}`).indexOf(`${id}`);
      this.messages.splice(idx, 1);

      // tracked property update via assignment
      this.messages = this.messages;
    }, HANG_TIME);
  }
}
```

`