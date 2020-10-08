# Activitypub API outline

Lemmy does not yet follow the ActivityPub spec in all regards. For example, we don't set a valid context indicating our context fields. We also ignore fields like `inbox`, `outbox` or `endpoints` for remote actors, and assume that everything is Lemmy. For an overview of deviations, read [#698](https://github.com/LemmyNet/lemmy/issues/698). They will be fixed in the near future.

In the following document, `Content` refers to either an ActivityPub `Page` or `Note`. `ContentAction` refers to any of `Create/Content`, `Update/Content`, `Like/Content`, `Dislike/Content`, `Remove/Content` or `Delete/Content`.

## Actors

### Community

Sends activities to user: `Accept/Follow`, `Announce/ContentAction`

Receives activities from user: `Follow`, `Undo/Follow`, `Create/Content`, `Update/Content`, `Like/Content`, `Dislike/Content`, `Remove/Content` (only admin/mod), `Delete/Content` (only creator), `Undo/ContentAction` (only for own actions)

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "http://enterprise.lemmy.ml/c/main",
    "type": "Group",
    "name": "main",
    "preferredUsername": "The Main Community",
    "category": { 
        "identifier": "1",
        "name": "Discussion"
    },
    "sensitive": false,
    "attributedTo": [
        "http://enterprise.lemmy.ml/u/picard",
        "http://enterprise.lemmy.ml/u/riker"
    ],
    "content": "Welcome to the default community!",
    "icon": {
        "type": "Image",
        "url": "http://enterprise.lemmy.ml/pictrs/image/Z8pFFb21cl.png"
    },
    "image": {
        "type": "Image",
        "url": "http://enterprise.lemmy.ml/pictrs/image/Wt8zoMcCmE.jpg"
    },
    "inbox": "http://enterprise.lemmy.ml/c/main/inbox",
    "outbox": "http://enterprise.lemmy.ml/c/main/outbox",
    "followers": "http://enterprise.lemmy.ml/c/main/followers",
    "endpoints": {
        "sharedInbox": "http://enterprise.lemmy.ml/inbox"
    },
    "published": "2020-10-06T17:27:43.282386+00:00",
    "updated": "2020-10-08T11:57:50.545821+00:00",
    "publicKey": {
        "id": "http://enterprise.lemmy.ml/c/main#main-key",
        "owner": "http://enterprise.lemmy.ml/c/main",
        "publicKeyPem": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEA9JJ7Ybp/H7iXeLkWFepg\ny4PHyIXY1TO9rK3lIBmAjNnkNywyGXMgUiiVhGyN9yU7Km8aWayQsNHOkPL7wMZK\nnY2Q+CTQv49kprEdcDVPGABi6EbCSOcRFVaUjjvRHf9Olod2QP/9OtX0oIFKN2KN\nPIUjeKK5tw4EWB8N1i5HOuOjuTcl2BXSemCQLAlXerLjT8xCarGi21xHPaQvAuns\nHt8ye7fUZKPRT10kwDMapjQ9Tsd+9HeBvNa4SDjJX1ONskNh2j4bqHHs2WUymLpX\n1cgf2jmaXAsz6jD9u0wfrLPelPJog8RSuvOzDPrtwX6uyQOl5NK00RlBZwj7bMDx\nzwIDAQAB\n-----END PUBLIC KEY-----\n"
    }
}
```

| Field Name | Mandatory | Description |
|---|---|---|
| `@context` | yes | The ActivityPub context |
| `id` | yes | Unique ID of this object |
| `type` | yes | Type of this object |
| `name` | yes | Name of the actor |
| `preferredUsername` | yes | Displayname |
| `category` | yes | Hardcoded list of categories, see https://dev.lemmy.ml/api/v1/categories |
| `sensitive` | yes | True indicates that all posts in the community are nsfw |
| `attributedTo` | yes | First the community creator, then all the remaining moderators |
| `content` | no | Text for the community sidebar, usually containing a description and rules |
| `icon` | no | Icon, shown next to the community name |
| `image` | no | Banner image, shown on top of the community page |
| `inbox` | no | ActivityPub inbox URL |
| `outbox` | no | ActivityPub outbox URL, only contains up to 20 latest posts, no comments, votes or other activities |
| `followers` | no | Follower collection URL, only contains the number of followers, no references to individual followers |
| `endpoints` | no | Contains URL of shared inbox |
| `published` | no | Datetime when the community was first created |
| `updated` | no | Datetime when the community was last changed |
| `publicKey` | yes | The public key used to verify signatures from this actor |
   
### User

Sends activities to Community: `Follow`, `Undo/Follow`, `Create/Content`, `Update/Content`, `Like/Content`, `Dislike/Content`, `Remove/Content` (only admin/mod), `Delete/Content` (only creator), `Undo/ContentAction` (only for own actions)

Receives activities from Community: `Accept/Follow`, `Announce/ContentAction`

Sends and receives activities from/to other users: All those related to private messages, namely `Create/Note`, `Update/Note`, `Delete/Note`, `Undo/Delete/Note`

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "http://enterprise.lemmy.ml/u/picard",
    "type": "Person",
    "name": "lemmy_alpha",
    "preferredUsername": "Jean-Luc Picard",
    "summary": "The user bio",
    "icon": {
        "type": "Image",
        "url": "http://enterprise.lemmy.ml/pictrs/image/DS3q0colRA.jpg"
    },
    "image": {
        "type": "Image",
        "url": "http://enterprise.lemmy.ml/pictrs/image/XenaYI5hTn.png"
    },
    "inbox": "http://enterprise.lemmy.ml/u/picard/inbox",
    "endpoints": {
        "sharedInbox": "http://enterprise.lemmy.ml/inbox"
    },
    "published": "2020-10-06T17:27:43.234391+00:00",
    "updated": "2020-10-08T11:27:17.905625+00:00",
    "publicKey": {
        "id": "http://enterprise.lemmy.ml/u/picard#main-key",
        "owner": "http://enterprise.lemmy.ml/u/picard",
        "publicKeyPem": "-----BEGIN PUBLIC KEY-----\nMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAyH9iH83+idw/T4QpuRSY\n5YgQ/T5pJCNxvQWb6qcCu3gEVigfbreqZKJpOih4YT36wu4GjPfoIkbWJXcfcEzq\nMEQoYbPStuwnklpN2zj3lRIPfGLht9CAlENLWikTUoW5kZLyU6UQtOGdT2b1hDuK\nsUEn67In6qYx6pal8fUbO6X3O2BKzGeofnXgHCu7QNIuH4RPzkWsLhvwqEJYP0zG\nodao2j+qmhKFsI4oNOUCGkdJejO7q+9gdoNxAtNNKilIOwUFBYXeZJb+XGlzo0X+\n70jdJ/xQCPlPlItU4pD/0FwPLtuReoOpMzLi20oDsPXJBvn+/NJaxqDINuywcN5p\n4wIDAQAB\n-----END PUBLIC KEY-----\n"
    }
}
```

| Field Name | Mandatory | Description |
|---|---|---|
| `@context` | yes | The ActivityPub context |
| `id` | yes | Unique ID of this object |
| `type` | yes | Type of this object |
| `name` | yes | Name of the actor |
| `preferredUsername` | yes | Displayname |
| `summary` | no | User bio |
| `icon` | no | The user's avatar, shown next to the username |
| `image` | no | The user's banner, shown on top of the profile |
| `inbox` | no | ActivityPub inbox URL |
| `endpoints` | no | Contains URL of shared inbox |
| `published` | no | Datetime when the user signed up |
| `updated` | no | Datetime when the user profile was last changed |
| `publicKey` | yes | The public key used to verify signatures from this actor |

## Objects

### Post

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://voyager.lemmy.ml/post/29",
    "type": "Page",
    "attributedTo": "https://voyager.lemmy.ml/u/picard",
    "to": "https://voyager.lemmy.ml/c/main",
    "summary": "Test thumbnail 2",
    "content": "blub blub",
    "url": "https://voyager.lemmy.ml:/pictrs/image/fzGwCsq7BJ.jpg",
    "image": {
        "type": "Image",
        "url": "https://voyager.lemmy.ml/pictrs/image/UejwBqrJM2.jpg"
    },
    "commentsEnabled": true,
    "sensitive": false,
    "stickied": false,
    "published": "2020-09-24T17:42:50.396237+00:00",
    "updated": "2020-09-24T18:31:14.158618+00:00",
}
```

| Field Name | Mandatory | Description |
|---|---|---|
| `@context` | yes | The ActivityPub context |
| `id` | yes | Unique ID of this object |
| `type` | yes | Type of this object |
| `attributedTo` | yes | ID of the user which created this post |
| `to` | yes | ID of the community where it was posted to |
| `summary` | yes | Title of the post |
| `content` | no | Body of the post |
| `url` | no | An arbitrary link to be shared |
| `image` | no | Thumbnail for `url`, only present if it is an image link |
| `commentsEnabled` | yes | False indicates that the post is locked, and no comments can be added |
| `sensitive` | yes | True marks the post as NSFW, blurs the thumbnail and hides it from users with NSFW settign disabled |
| `stickied` | yes | True means that it is shown on top of the community |
| `published` | no | Datetime when the post was created |
| `updated` | no | Datetime when the post was edited (not present if it was never edited) |

### Comment

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "https://enterprise.lemmy.ml/comment/95",
    "type": "Note",
    "attributedTo": "https://enterprise.lemmy.ml/u/picard",
    "to": "https://enterprise.lemmy.ml/c/main",
    "content": "mmmk",
    "inReplyTo": [
        "https://enterprise.lemmy.ml/post/38",
        "https://voyager.lemmy.ml/comment/73"
    ],
    "published": "2020-10-06T17:53:22.174836+00:00",
    "updated": "2020-10-06T17:53:22.174836+00:00"
}
```

| Field Name | Mandatory | Description |
|---|---|---|
| `@context` | yes | The ActivityPub context |
| `id` | yes | Unique ID of this object |
| `type` | yes | Type of this object |
| `attributedTo` | yes | ID of the user who created the comment |
| `to` | yes | Community where the comment was made |
| `content` | yes | The comment text |
| `inReplyTo` | yes | IDs of the post where this comment was made, and the parent comment. If this is a top-level comment, `inReplyTo` only contains the post |
| `published` | no | Datetime when the comment was created |
| `updated` | no | Datetime when the comment was edited (not present if it was never edited) |

### Private Message

```json
{
    "@context": "https://www.w3.org/ns/activitystreams",
    "id": "http://enterprise.lemmy.ml/private_message/34",
    "type": "Note",
    "attributedTo": "http://enterprise.lemmy.ml/u/picard",
    "to": "http://voyager.lemmy.ml/u/janeway",
    "content": "test",
    "published": "2020-10-08T19:10:46.542820+00:00",
    "updated": "2020-10-08T20:13:52.547156+00:00",
}
```

| Field Name | Mandatory | Description |
|---|---|---|
| `@context` | yes | The ActivityPub context |
| `id` | yes | Unique ID of this object |
| `type` | yes | Type of this object |
| `attributedTo` | ID of the user who created this private message |
| `to` | ID of the recipient |
| `content` | yes | The text of the private message |
| `published` | no | Datetime when the message was created |
| `updated` | no | Datetime when the message was edited (not present if it was never edited) |

## Activities

### Follow/Unfollow

### Create

### Update

### Like

### Dislike

### Remove

### Delete

### Announce

### Undo
