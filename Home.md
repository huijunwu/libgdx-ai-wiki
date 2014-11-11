![gdxai logo](https://cloud.githubusercontent.com/assets/2366334/4677025/64ae592a-55e2-11e4-8a31-31c2941ff995.png)


## What is gdxAI?

It's an artificial intelligence framework, entirely written in Java, for game development with [libGDX](https://github.com/libgdx/libgdx).

The gdxAI framework makes use of a limited number of classes from the libGDX framework, mainly collections which are optimized for mobile platforms by limiting garbage creation and supporting primitive types directly, so avoiding boxing and unboxing. However, the gdxAI framework does not force you to develop your application using the libGDX framework if you do not wish to do so, but the libGDX jar remains an essential requirement.

GdxAI tries to be a high-performance framework providing some of the most common AI techniques used by game industry.
However, in the present state of the art, the gdxAI framework covers only part of the entire game AI area, which is really huge. We've tried to focus on what matters most in game AI development, though. And more stuff will come soon.

## About this wiki

Throughout this wiki, we will cover the entirety of the gdxAI framework, its fundamentals, the theory behind it and how to design intelligent agents.

This is a manual, for a comprehensive API reference, check the [official javadocs](http://libgdx.badlogicgames.com/gdx-ai/docs/).

## Technical considerations

GdxAI is compatible with Java 6 and [GWT](http://www.gwtproject.org/), which means you can also target the browser.

Thanks to its [Apache 2.0](https://github.com/libgdx/gdx-ai/blob/master/LICENSE) license, users have the freedom to use it, modify it and redistribute as they please. Commercial purposes are perfectly allowed.

We're continuously testing and building GdxAI with Jenkins.

[![Build Status](http://144.76.220.132:8080/job/gdx-ai/badge/icon)](http://144.76.220.132:8080/job/gdx-ai/)

