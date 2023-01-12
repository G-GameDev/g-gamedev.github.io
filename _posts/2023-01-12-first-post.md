---
title: "First blog post!"
date: 2023-01-12T15:51:00-01:00
categories:
  - blog
  - C++
  - Unreal Engine
  - Blueprints
tags:
  - Jekyll
  - update
---

You'll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. You can rebuild the site in many different ways, but the most common way is to run `jekyll serve`, which launches a web server and auto-regenerates your site when a file is updated.

To add new posts, simply add a file in the `_posts` directory that follows the convention `YYYY-MM-DD-name-of-post.ext` and includes the necessary front matter. Take a look at the source for this post to get an idea about how it works.

Jekyll also offers powerful support for code snippets:

```cpp
void ADungeonFloor::OpenAllDoors()
{
	if (FloorTiles[GetAbsIdxFromRowCol(1, 2)]->DoorTop)
	{
		FloorTiles[GetAbsIdxFromRowCol(1, 2)]->DoorTop->ShowOpening();
	}

	/*for (ADungeonDoor* Door : Doors)
	{
		Door->ShowOpening();
	}*/
}

void ADungeonFloor::CloseAllDoors()
{
	for (ADungeonDoor* Door : Doors)
	{
		Door->ShowClosing();
	}
}
```

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
