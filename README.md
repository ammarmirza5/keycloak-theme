# Sabthings ID — Keycloak Theme

Custom Keycloak login theme used by all Sabthings products (Khel Maidan, Spring backend, future apps). Branded as **Sabthings ID** — one account, every app.

This folder is the source of truth. The live theme on `auth.sabthings.com` is whatever was last deployed from here.

---

## Folder layout

```
keycloak-theme/
└── sabthings/                              ← theme name (matches the dropdown in Keycloak admin)
    └── login/
        ├── theme.properties                ← declares parent = keycloak.v2
        ├── resources/
        │   ├── css/sabthings.css           ← all styling overrides
        │   └── img/logo.svg                ← Sabthings wordmark (replace with official asset)
        └── messages/
            └── messages_en.properties      ← reworded strings (Sabthings ID branding)
```

To add other languages later: drop `messages_ur.properties`, `messages_ar.properties`, etc. into `messages/`, and uncomment the `locales=` line in `theme.properties`.

---

## Deploying to Dokploy

Dokploy runs Keycloak in a Docker container. You need to mount this `sabthings/` folder into `/opt/keycloak/themes/sabthings/` inside the container.

### Step 1 — Upload theme files to the Dokploy server

SSH into the server hosting Dokploy and create a folder for the theme:

```bash
ssh root@your-dokploy-host
mkdir -p /var/dokploy/keycloak-themes
cd /var/dokploy/keycloak-themes
# Then upload the sabthings/ folder here, e.g. via scp/rsync from your laptop:
#   rsync -av --delete keycloak-theme/sabthings/ root@host:/var/dokploy/keycloak-themes/sabthings/
```

End state on the server:
```
/var/dokploy/keycloak-themes/sabthings/login/theme.properties
/var/dokploy/keycloak-themes/sabthings/login/resources/css/sabthings.css
/var/dokploy/keycloak-themes/sabthings/login/resources/img/logo.svg
/var/dokploy/keycloak-themes/sabthings/login/messages/messages_en.properties
```

### Step 2 — Add a volume mount in Dokploy

In the Dokploy UI:

1. Open your **Keycloak** application
2. Go to the **Mounts** (or **Volumes**) tab
3. Add a new bind mount:

   | Field | Value |
   |---|---|
   | **Type** | Bind mount |
   | **Host path** | `/var/dokploy/keycloak-themes/sabthings` |
   | **Container path** | `/opt/keycloak/themes/sabthings` |
   | **Read only** | ✓ checked (recommended) |

4. Save and **redeploy** the Keycloak service so the mount takes effect.

### Step 3 — Activate the theme in Keycloak admin

1. Open `https://auth.sabthings.com`
2. **Realm settings → Themes** tab
3. **Login theme:** select `sabthings` from the dropdown
4. (Optional) **Account theme** + **Email theme** can stay on `keycloak.v2` for now — they're separate
5. **Save**

Hard-reload the login page (Ctrl+Shift+R). You should see the Sabthings wordmark, navy primary button, and "Sign in to your Sabthings ID" heading.

If `sabthings` is **not** in the dropdown:
- The bind mount didn't pick up. Check `docker exec <keycloak-container> ls /opt/keycloak/themes/` — `sabthings` should appear alongside `base`, `keycloak`, `keycloak.v2`.
- Restart the Keycloak container after fixing the mount.

---

## Updating the theme

| What you changed | What to do |
|---|---|
| `messages_en.properties` only | Push to server (`rsync`) — no restart. New text shows on next page load. |
| `sabthings.css` only | Push to server — no restart. Hard-reload browser (Ctrl+Shift+R) to bypass cache. |
| `logo.svg` (same filename) | Push — no restart. Hard-reload browser. |
| `theme.properties` | Push + **restart the Keycloak container** in Dokploy. |
| Added a new file (e.g. a template override) | Push + restart. |

Pro tip: in Keycloak admin → **Realm settings → General**, set `Frontend URL` to `https://auth.sabthings.com` and turn **off** `Internationalization` until you actually add a second language — otherwise Keycloak shows a language dropdown with only English in it.

---

## Replacing the placeholder logo

Open `sabthings/login/resources/img/logo.svg`. The current contents are a generic blue gradient mark + "Sabthings ID" wordmark. Replace with the real Sabthings logo:

- **Recommended format:** SVG (scales perfectly, ~2 KB)
- **Recommended dimensions:** 320×64 viewBox (matches the CSS height of 64px in `#kc-header-wrapper`)
- **Keep the filename `logo.svg`** so no CSS change is needed
- If you must use PNG, save as `logo.png` and change the line in `sabthings.css`:
  ```css
  background-image: url('../img/logo.png');
  ```
  Use 2x or 3x resolution (e.g. 640×128) so it stays crisp on retina screens.

---

## Customising further

The theme **inherits** from `keycloak.v2`, so you only override what you want. The fastest way to find what to override:

1. Open the live login page in browser DevTools
2. Inspect any element — note its CSS class (e.g. `.pf-v5-c-button.pf-m-primary`)
3. Add a rule for that class to `sabthings.css`
4. Push to server

For full template overrides (changing HTML structure, not just CSS), copy the matching `.ftl` file from Keycloak's source:
- <https://github.com/keycloak/keycloak/tree/main/themes/src/main/resources/theme/keycloak.v2/login>

…into `sabthings/login/` (same filename). Keycloak prefers your version over the parent's. This is rare — CSS is usually enough.

---

## Why a custom theme (not just realm settings)

Realm settings (display name, registration on/off, etc.) cover words and toggles but **cannot change visual identity**. The default `keycloak.v2` theme is generic Red Hat / Keycloak branding — fine for internal tools, wrong for a customer-facing identity service called "Sabthings ID".

Owning the theme also means:
- One coherent brand across every app that authenticates here
- Translations live in one place (one `messages_ur.properties` covers Khel Maidan + Spring app + everything else)
- Future apps just point their Keycloak client at the same realm and inherit the styling for free — no per-app login screen to design

---

## Related

- [docs/21_Keycloak_Integration.md](../docs/21_Keycloak_Integration.md) — backend integration (Laravel guards, JWKS, user sync, OIDC flows)
- Keycloak theme reference: <https://www.keycloak.org/docs/latest/server_development/#_themes>
- Default `keycloak.v2` source (use as reference for class names): <https://github.com/keycloak/keycloak/tree/main/themes/src/main/resources/theme/keycloak.v2>
