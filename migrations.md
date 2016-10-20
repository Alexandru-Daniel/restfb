# Migration Guide
Because we have a lot of API changes from version 1.x to 2.0 RestFB provides a migration guide, where the breaking changes can be found

## tl;dr
* Logging changed, `slf4j` is used if present
* `Comment.Attachment` -> `StoryAttachment`
* `Comment.isCanComment` -> `Comment.getCanComment`
* `Comment.isCanHide` -> `Comment.getCanHide`
* `FeedPostValue.isHidden` -> `FeedPostValue.getIsHidden`
* `Message.isRenderAsSticker` -> `Message.getRenderAsSticker`
* `FacebookClient.executeQuery()` -> `FacebookClient.executeFqlQuery()`
* `FacebookClient.executeMultiquery()` -> `FacebookClient.executeFqlMultiquery()`
* `User.Picture` -> `ProfilePictureSource`
* `Event.getPicture()` return type `String` -> `Event.getPicture()` return type `ProfilePictureSource`
* `Event.getVenue()` return type `Venue` -> returns `Location` now
* `Group.getVenue()` return type `Venue` -> returns `Location` now
* `Checkin.Place` replaced with `Place` type.
* `Post.Place` replaced with `Place` type.
* `Post.Likes` replaced with `Likes` type.
* `Post.Comments` replaced with `Comments` type.
* `Post.Privacy` replaced with `Privacy` type.
* `Photo.getComments` returns `Comments` type instead of `Comment list
* json API changed, you should look [here](https://github.com/ralfstx/minimal-json)
* `Post.MessageTag` replaced with `MessageTag` type
* `Likes` data type moved from `NamedFacebookType` to `LikeItem`
* `likes` field changed from `long` to `Likes` in `Page` type
* `likesCount` field added to `Page` type

## Logging changed
As soon as the `slf4j` jars are found on the classpath, slf4j is used 
as logging framework. As fallback we use the `java.util.logging` 
framework. In the rare case you have `slf4j` on the classpath, but you 
don't want RestFB to use it, you can switch to the `java.util.logging` with 
the system.property "`com.restfb.forceJUL=true`". 

Don't forget to add a RestFB logging configuration to your logging config.

## Comment.Attachment removed
We have removed the `Comment.Attachment` type because it is replaced by 
a newer type called `StoryAttachment`. `StoryAttachment` has the same 
fields as `Comment.Attachment` but contains the target field, too. 
Therefore it is better suitable to the original Facebook JSON.

## `boolean` changed to `Boolean`
Facebook returns for Graph API 2.4+ only requested fields. So some fields are
not set. Because `boolean` is only `true` or `false` it is changed to Boolean. So
a field can be `null` if it is not set. A developer needs to check for null now,
but get better results if a field was not requested and not tricked by the Java
default value for `boolean`.

## `FacebookClient#executeQuery` and `FacebookClient#executeMultiquery` removed
The `FacebookClient` interface does not longer provide the `executeQuery` and 
`executeMultiquery` access to FQL. These methods are not longer working on the
Facebook side, so we don't need to support them further.
The `DefaultFacebookClient` does not longer contain these methods. Because some
developer still use the FQL interface (Graph API 2.0) they have to switch to the
newer (and working) `executeFqlQuery` and `executeFqlMultiquery`.

## User.Picture removed
We had to remove the `User.Picture` type because we drop the
compatibility. The picture field is now the `ProfilePictureSource`
type.

## Event.getPicture() returns `ProfilePictureSource` instead of `String`
The `Event.getPicture()` returned the image url as `String`. We changed this because
Facebook provides more information than only a `String`. The `ProfilePictureSource` is
a better choice.

## `Venue` replaced with `Location` type
The `Group` and the `Event` type contain a venue but this is the same type as `Location`.
So we replaced it and don't need to take care of two different java entities.

## deprecated classes removed
We have some types that are only left, because we didn't want the break the API
before version 2.0. With the new major release we can remove them and do some
cleanup. 

* `Checkin.Place` replaced with `Place` type.
* `Post.Place` replaced with `Place` type.
* `Post.Likes` replaced with `Likes` type.
* `Post.Comments` replaced with `Comments` type.
* `Post.Privacy` replaced with `Privacy` type.`

## The `photo` type returns `Comments` type instead of list
Because the comments of a photo not only contains the comments we have to change the
returned type. Now you may access the summary object and fetch the total count.

## Json.org parser replaced with minimal-json
The Json.org parser is not as free as it should be and so we replaced
it with the minimal-json parser. It's published under the MIT license
and provides a much better performance. The minimal json benchmarks are
very promising and our own benchmarks in the restfb benchmark project
support their statements. Because minimal json has a different API you have
to change your code accordingly. To understand the API better, you should have
a look at the [github repository of minimal json](https://github.com/ralfstx/minimal-json).

## `Post.MessageTag` replaced with `MessageTag`
The `Post` object contains some inner classes. We removed the inner class `MessageTag` and replaced
it with the `MessageTag` class we introduced already. The inner class was only for compatibility reasons there.
If you work with MessageTags you have to change your import.

## `Likes` type data type changed
The `Likes` type contains a list of NamedFacebookTypes. These elements are in some cases not the correct type.
So we changed the type and introduces the new `LikeItem` type. The new `LikeItem` type contains the `created_time` field.
Most of the time this value is not set, but we provide it nevertheless.

## `Likes` added to `Page` type
With Graph API 2.6 the `likes` field does not contain the count of the likes. Instead it is
filled with the a list of likes. Therefore the returned type is `Likes`. Because in earlier
versions we should be able to access the field as long, we use the duplicate mapping magic and
renamed the old `likes` field to `likesCount`. This is similar to the `fanCount` that was introduced
with Graph API 2.6 for the `Page` object.