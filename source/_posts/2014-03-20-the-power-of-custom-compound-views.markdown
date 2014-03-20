---
layout: post
title: "The power of custom compound views"
date: 2014-03-20 07:49
comments: true
author: "Daniel Grech"
categories: [Android, UI]
---

As an app grows in features and complexity, the amount of code it takes to customise and polish the UI increases. Custom views can be a powerful tool in cleaning up your code and making it generally more pleasureable to read. Separating your view presentation code from your business logic (say, in your activity) is one of the basic tenants of [MVC](http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller). 

Let's look at a simple example of where this customisation can lead to bloated code. Consider populating a ListView with some items in your adapter. Too often do I see code like this:

``` java Base model class to present
public class Person {
    int mId;
    String mName;
    String mAddress;
    int mAge;
    Uri mAvatarUri;
}
```

``` xml Your list item layout - res/layout/li_person.xml
<LinearLayout
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <ImageView
            android:id="@+id/image"
            android:layout_width="64dp"
            android:layout_height="64dp"/>
    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
        <TextView
                android:id="@+id/name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
        <TextView
                android:id="@+id/address"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
        <TextView
                android:id="@+id/age"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
    </LinearLayout>
</LinearLayout>
```

``` java Bloated adapter implementation
public class MyListAdapter extends BaseAdapter {
    private List<Person> mPersons;

    public MyListAdapter(List<Person> persons) {
    	mPersons = persons;
    }

    @Override
    public int getCount() {
        return mPersons == null ? 0 : mPersons.size();
    }

    @Override
    public Person getItem(int position) {
        return mPersons.get(position);
    }

    @Override
    public long getItemId(int position) {
        return getItem(position).mId;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        final ViewHolder holder;
        if (convertView == null) {
            convertView = LayoutInflater.from(parent.getContext())
                    .inflate(R.layout.li_person, parent, false);

            holder = new ViewHolder();
            holder.mNameView = (TextView) convertView.findViewById(R.id.name);
            holder.mAddressView = (TextView) convertView.findViewById(R.id.address);
            holder.mAgeView = (TextView) convertView.findViewById(R.id.age);
            holder.mImageView = (ImageView) convertView.findViewById(R.id.image);

            convertView.setTag(holder);
        } else {
            holder = (ViewHolder) convertView.getTag();
        }

        final Person p = getItem(position);

        holder.mNameView.setText(p.mName == null ? null : "Unknown");
        holder.mAddressView.setText(p.mAddress == null ? null : "Unknown");
        holder.mAgeView.setText(p.mAge < 0 ?  "Unknown" : String.valueOf(p.mAge));

        if (p.mAvatarUri == null) {
            holder.mImageView.setVisibility(View.GONE);
        } else {
            holder.mImageView.setVisibility(View.VISIBLE);
            holder.mImageView.setImageURI(p.mAvatarUri);
        }

        return convertView;
    }

	// @See http://developer.android.com/training/improving-layouts/smooth-scrolling.html
	private static class ViewHolder {
	    TextView mNameView;
	    TextView mAddressView;
	    TextView mAgeView;
	    ImageView mImageView;
	}
}	
```

This pattern is fine for the most basic use-cases (such as displaying a single TextView for each list item), but even in this small example there is a lot of presentation-specific cruft. Imagine a more complex view (remote image loading, date formatting, animation etc) and this code can quickly grow out of control!

Show what can we do about this? Encapsulate all of this logic in a custom view class!

<!-- more -->

Not only will using a custom view help this keep your activity code clean, it also encourages code reuse and removes all the ViewHolder boilerplate nobody likes to write.

First, update your xml layout:

``` xml res/layout/li_person.xml
<com.example.app.view.PersonListItemView
        xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    <ImageView
            android:id="@+id/image"
            android:layout_width="64dp"
            android:layout_height="64dp"/>
    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
        <TextView
                android:id="@+id/name"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
        <TextView
                android:id="@+id/address"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
        <TextView
                android:id="@+id/age"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"/>
    </LinearLayout>
</com.example.app.view.PersonListItemView>
```

Note that we modified our root layout element to point to our custom view, defined below:

``` java Custom compound view
public class PersonListItemView extends LinearLayout {

    TextView mNameView;

    TextView mAddressView;

    TextView mAgeView;

    ImageView mImageView;

    public static PersonListItemView inflate(ViewGroup parent) {
        return (Person) LayoutInflater.from(parent.getContext())
                .inflate(R.layout.li_person, parent, false);
    }

    public PersonListItemView(Context context) {
        super(context);
    }

    public PersonListItemView(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public PersonListItemView(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
    }

    @Override
    protected void onFinishInflate() {
        super.onFinishInflate();

        mNameView = (TextView) findViewById(R.id.name);
        mAddressView = (TextView) findViewById(R.id.address);
        mAgeView = (TextView) findViewById(R.id.age);
        mImageView = (ImageView) findViewById(R.id.image);
    }

    public void populate(Person p) {
        mNameView.setText(p.mName == null ? null : "Unknown");
        mAddressView.setText(p.mAddress == null ? null : "Unknown");
        mAgeView.setText(p.mAge < 0 ?  "Unknown" : String.valueOf(p.mAge));

        if (p.mAvatarUri == null) {
            mImageView.setVisibility(View.GONE);
        } else {
            mImageView.setVisibility(View.VISIBLE);
            mImageView.setImageURI(p.mAvatarUri);
        }
    }
}

```

Well would you look at that - all our presentation logic is where it should be - completely hidden in the view!

All that's left now is to update our adapter:

``` java Cleaner adapter implementation
public class MyListAdapter extends BaseAdapter {

    private List<Person> mPersons;

    public MyListAdapter(List<Person> persons) {
        mPersons = persons;
    }

    @Override
    public int getCount() {
        return mPersons == null ? 0 : mPersons.size();
    }

    @Override
    public Person getItem(int position) {
        return mPersons.get(position);
    }

    @Override
    public long getItemId(int position) {
        return getItem(position).mId;
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        PersonListItemView personView = (PersonListItemView) convertView;
        if (personView == null) {
            personView = PersonListItemView.inflate(parent);
        }

        final Person p = getItem(position);
        personView.populate(p);

        return personView;
    }
}
```

How easy was that! Our adapter has been greatly simplified and can now focus on it's primary concern - mapping our underlying data to our view. Nothing more, nothing less :)

This is just a small example of where this kind of custom view can be useful and is by no means limited to just ListView items. You can apply this to any old view - it's sometimes useful to define your whole activity/fragment view this way, too. Anything you can do to keep your underlying controller class clean, simple and focused.

Happy coding!
