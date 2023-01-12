---
title: "First blog post!"

date: 2023-01-12T15:51:00+01:00

categories:
  - blog
  - C++
  - Blueprints

tags:
  - Unreal Engine

---

This is the start of the GGameDev blog.
I want to use this site to tie together the game-dev work I do, with the YouTube channel, 
and provide some 'text based' reference to accompany those videos, 
and documentation for things like the plugins I create.
(Not everyone likes to refer to videos, some people prefer a more 'traditional' documentation approach.

{% highlight cpp linenos %}
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
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
