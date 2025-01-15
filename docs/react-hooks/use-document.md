---
sidebar_position: 2
---

# useDocument

The `useDocument` hook allows you to subscribe to and modify documents in your Fireproof database. When sync is enabled, multiple users can update the same document in real-time. For complex collaborative editing scenarios, consider using operational transform libraries like [Yjs](https://github.com/yjs/yjs) or [Automerge](https://automerge.org), which work seamlessly with Fireproof.

## Basic Usage

Acquire `useDocument` from the `useFireproof` hook. Here's a simple example:

```js
import { useFireproof } from 'use-fireproof';

const { useDocument } = useFireproof("my-app")
const [doc, setDoc, saveDoc] = useDocument(() => ({_id: "user-123"}));
```

The hook returns three items:
- `doc`: The current document state
- `setDoc`: Function to update the document state
- `saveDoc`: Function to persist changes to the database

Initially, the returned document is the initial value passed to `useDocument`. The component will re-render when the document loads or when the document is updated in the database.

While the `_id` field is the only field required to retrieve the document, if you are using TypeScript you will likely need to craft an empty object with the correct `_id` and wait for the document to load. See the Updating example below.

## Creating New Documents

To create a new document, use `useDocument` without specifying an `_id`. The database will automatically assign one when you call `saveDoc`. Here's a real-world example:

```typescript
function CreateUserProfile() {
  const { useDocument } = useFireproof("user-app");
  const [user, setUser, saveUser] = useDocument(() => ({
    type: 'user',
    firstName: '',
    lastName: '',
    email: '',
    role: 'member',
    active: true,
    updatedAt: new Date().toISOString(),
    createdAt: new Date().toISOString()
  }));

  const handleCreate = async () => {
    const result = await saveUser(user);
    // result.id contains the new document ID
  };

  return (
    <form onSubmit={handleCreate}>
      <input
        value={user.firstName}
        onChange={(e) => setUser({ firstName: e.target.value })}
        placeholder="First Name"
      />
      <button type="submit">Create Profile</button>
    </form>
  );
}
```

## Updating Existing Documents

When updating documents, `useDocument` will automatically handle state management and persistence. Here's an example of a profile editor:

```typescript
function UserProfileEditor({ userId }) {
  const { useDocument } = useFireproof("user-app");
  const [user, setUser, storeUser] = useDocument(() => ({
    _id: userId,
    type: 'empty',
  }));

  const updateProfile = async (updates) => {
    const updated = {
      ...user,
      ...updates,
      updatedAt: new Date().toISOString()
    };
    setUser(updated);
  };

  const saveProfile = async () => {
    const result = await storeUser(user);
    // result.id contains the updated document ID
  };

  return user.type !== 'empty' && (
    <div>
      <input
        value={user.firstName}
        onChange={(e) => updateProfile({ firstName: e.target.value })}
        placeholder="First Name"
      />
      <input
        value={user.lastName}
        onChange={(e) => updateProfile({ lastName: e.target.value })}
        placeholder="Last Name"
      />
      <input
        value={user.email}
        onChange={(e) => updateProfile({ email: e.target.value })}
        placeholder="Email"
      />
      <select
        value={user.role}
        onChange={(e) => updateProfile({ role: e.target.value })}
      >
        <option value="admin">Admin</option>
        <option value="member">Member</option>
        <option value="guest">Guest</option>
      </select>
      <label>
        <input
          type="checkbox"
          checked={user.active}
          onChange={(e) => updateProfile({ active: e.target.checked })}
        />
        Account Active
      </label>
      <button onClick={saveProfile}>Save</button>
    </div>
  );
}
```
