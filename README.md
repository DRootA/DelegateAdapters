# DelegateAdapters
Simplify creating recycler view adapters with many different view types.
This lib is inspired by Hannes Dorfmann [AdapterDelegates](https://github.com/sockeqwe/AdapterDelegates)

## Dependencies

```groovy
compile 'com.github.liverm0r:delegateadapters:v1.06'
```

You also have to add this in your project build.gradle

```groovy
allprojects {
    repositories {
        ...
        maven { url 'https://jitpack.io' }
    }
}
```

[![Build Status](https://travis-ci.org/sockeqwe/AdapterDelegates.svg?branch=master)](https://jitpack.io/#Liverm0r/delegateadapters)


## Example of usage in Java

Inherit from BaseDelegateAdapter:

```java
public class TextDelegateAdapter
    extends BaseDelegateAdapter<TextDelegateAdapter.TextViewHolder, TextViewModel> {


    @Override
    protected void onInflated(@NonNull View view,
                              @NonNull TextViewModel item,
                              @NonNull TextViewHolder viewHolder) {
        viewHolder.tvTitle.setText(item.title);
        viewHolder.tvDescription.setText(item.description);
    }

    @Override
    protected int getLayoutId() {
        return R.layout.text_item;
    }

    @Override
    protected TextViewHolder createViewHolder(View parent) {
        return new TextViewHolder(parent);
    }

    @Override
    public boolean isForViewType(@NonNull List<Object> items, int position) {
        return items.get(position) instanceof TextViewModel;
    }


    final class TextViewHolder extends BaseViewHolder {

        private TextView tvTitle;
        private TextView tvDescription;

        private TextViewHolder(View parent) {
            super(parent);
            tvTitle = parent.findViewById(R.id.tv_title);
            tvDescription = parent.findViewById(R.id.tv_description);
        }
    }
}

```

TextViewModel is just a POJO:

```java
public class TextViewModel {

    @NonNull public final String title;
    @NonNull public final String description;

    public TextViewModel(@NonNull String title, @NonNull String description) {
        this.title = title;
        this.description = description;
    }
}
```

Now you can use CompositeDelegateAdapter just like base RecyclerView.Adapter, composing it with whatever amount of delegate adapters:

```java

    CompositeDelegateAdapter adapter = new CompositeDelegateAdapter.Builder()
        .add(new ImageDelegateAdapter(onImageClick))
        .add(new TextDelegateAdapter())
        .add(new CheckDelegateAdapter())
        .build();
    recyclerView.setLayoutManager(new LinearLayoutManager(this));
    recyclerView.setAdapter(adapter);
```

In current example you will just have three types - text, image, and checkbox

![example](https://github.com/Liverm0r/DelegateAdapters/blob/master/feed_example.jpg)

## Example of usage in Kotlin


With Kotlin delegate adapters become much smaller:

```kotlin

class ImageDelegateAdapter(private val onImageClick: (ImageViewModel) -> Unit)
    : KDelegateAdapter<ImageViewModel>() {

    override fun onInflated(item: ImageViewModel, viewHolder: KViewHolder)= with(viewHolder) {
        tv_title.text = item.title
        img_bg.setOnClickListener { onImageClick(item) }
        img_bg.setImageResource(item.imageRes)
    }

    override fun isForViewType(items: List<Any>, position: Int) = items[position] is ImageViewModel

    override fun getLayoutId(): Int = R.layout.image_item
}

```

Check part `wiht(viewHolder)`. This works like basic view holder without creating one. See the [View holder pattern support and caching options](
https://github.com/Kotlin/KEEP/blob/master/proposals/android-extensions-entity-caching.md
) for more information.

To enable this, you need to turn on kotlin ext. experimental feature in your module build.gradle file:

```groovy
androidExtensions {
    experimental = true
}
```

After that you will be able to write some basic kotlin delegate adapter class, like

```kotlin
abstract class KDelegateAdapter<T> : BaseDelegateAdapter<KDelegateAdapter.KViewHolder, T>() {

    abstract fun onInflated(item: T, viewHolder: KViewHolder)

    final override fun onInflated(view: View, item: T, viewHolder: KViewHolder) {
        onInflated(item, viewHolder)
    }

    override fun createViewHolder(parent: View?): KViewHolder {
        return KViewHolder(parent)
    }

    class KViewHolder(override val containerView: View?) : BaseViewHolder(containerView), LayoutContainer
}
```
Check the examples in this project.

  ## License

```
Copyright 2017 Artur Dumchev 

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

   http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```
