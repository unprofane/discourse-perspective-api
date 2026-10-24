# Discourse Perspective plugin, with the host as a setting

This is [discourse/discourse-perspective-api](https://github.com/discourse/discourse-perspective-api) with one change: a site setting, `perspective_api_host`, for the address the plugin calls. Google's Perspective API stops answering after **December 31, 2026**, and the official plugin posts to Google's host with no way to change it. Maintained by [Unprofane](https://unprofane.com); the rest of the plugin is Discourse's, under the same GNU GPL v2.

## Switching host

1. Install this repository instead of Discourse's: in `app.yml`, under `hooks: after_code:`, replace `https://github.com/discourse/discourse-perspective-api.git` with `https://github.com/unprofane/discourse-perspective-api.git`, then rebuild (`./launcher rebuild app`).
2. In the admin site settings, search for `perspective api host` and set it to a Perspective-compatible host, such as `https://api.unprofane.com` ([Unprofane's drop-in](https://unprofane.com/perspective/discourse/)), and put that host's key in `perspective google api key`.

Left blank or at its default, the setting keeps Google's host, so the plugin behaves exactly as Discourse's does. The scores keep their meaning: the thresholds, the flags, the composer warning and the backfill custom fields are unchanged.

Discourse's own hosting runs only the plugins Discourse ships, so a forum hosted by Discourse cannot install this fork; a self-hosted forum, or a host that installs plugins from a repository you name, can. [The guide](https://unprofane.com/perspective/discourse/) covers each case.

---

# Discourse Perspective plugin (upstream README)

https://meta.discourse.org/t/google-perspective-api-plugin/98733

This plugin flags toxic posts using Google's Perspective API.

## Installation

Follow the directions at [Install a Plugin](https://meta.discourse.org/t/install-a-plugin/19157) using https://github.com/discourse/discourse-perspective-api.git as the repository URL.

## Authors

Erick Guan

## License

GNU GPL v2

## Data Explorer Queries

If you choose standard mode, use `post_perspective_toxicity`. Otherwise, replace them to `post_perspective_severe_toxicity`. For most toxic categories and users, I choose a
probability 0.85 as the threshold.

Most toxic categories:

```sql
SELECT COUNT(pc) as counts, c.id, c.name, c.description
FROM post_custom_fields pc
JOIN posts p ON p.id = pc.post_id
JOIN topics t ON t.id = p.topic_id
JOIN categories c ON c.id = t.category_id
WHERE pc.name = 'post_perspective_toxicity' AND pc.value >= '0.85'
GROUP BY c.id
ORDER BY counts DESC
LIMIT 100
```

Most toxic users:

```sql
SELECT COUNT(pc) as counts, u.id, u.username, u.trust_level, u.suspended_till, u.silenced_till
FROM post_custom_fields pc
JOIN posts p ON p.id = pc.post_id
JOIN users u ON p.user_id = u.id
WHERE pc.name = 'post_perspective_toxicity' AND pc.value >= '0.85'
GROUP BY u.id
ORDER BY counts DESC
LIMIT 100
```

Most toxic posts:

```sql
SELECT p.id, pc.value, p.user_id, p.topic_id, p.created_at, p.raw
FROM post_custom_fields pc
JOIN posts p ON p.id = pc.post_id
WHERE pc.name = 'post_perspective_toxicity'
ORDER BY pc.value DESC
LIMIT 100
```

Most toxic posts today:

```sql
SELECT p.id, pc.value, p.user_id, p.topic_id, p.created_at, p.raw
FROM post_custom_fields pc
JOIN posts p ON p.id = pc.post_id
WHERE pc.name = 'post_perspective_toxicity' AND
p.created_at >= CURRENT_DATE
ORDER BY pc.value DESC
LIMIT 100
```
