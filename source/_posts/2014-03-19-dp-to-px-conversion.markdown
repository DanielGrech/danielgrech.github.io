---
layout: post
title: Dp to Px conversion
date: 2014-03-19 19:08
comments: true
author: Daniel Grech
categories: Android
---

There are often times doing Android development when you will need to apply some dimension value directly in your java code. Anyone creating Android UI's should be familiar with the concept of [Display-independant pixels](http://developer.android.com/training/multiscreen/screendensities.html#TaskUseDP). This is most commonly seen in you layout xml definitions.

Sometimes, there is a need to calculate and apply these same dimen values dynamically in your app. Naively, you could try this:

``` java Padding without taking into account display density
public class MyAwesomeActivity extends Activity {
	
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		// ...
		
		View view = findViewById(R.id.my_awesome_view);
		
		// We want to apply 20dp of padding to the view. Simple, no?
		view.setPadding(20, 20, 20, 20);
		
		// ...
	}	
}
```

The problem with the above snippet is that when applying dimens in your java code, all values are assumed to be in pixel values! This means that things will work fine on an mdpi device (which have a 1 - 1 ratio between px and dp), but on any screens with a higher density, the padding will be smaller than intended. For example, on an xhdpi screen (with a 2 - 1 px to dp ratio), the padding will only be equivalent to 10dp!

To fix this, we need to convert our intended dp value into the correct number of pixels for the current device.
<!-- more -->
Luckily Android gives us the tools to do this!

``` java Convert dps to pixels
public static float dpToPixels(Context context, int dps) {
	return TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, dps,
		context.getResources().getDisplayMetrics())
}

```

With this utility method, it's simply a case of:

``` java Dynamic padding which properly taking into account display metrics
public class MyAwesomeActivity extends Activity {
	
	@Override
	public void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		
		// ...
		
		View view = findViewById(R.id.my_awesome_view);
		
		// Hooray - padding will now be 20dp on all devices
		int p = dpToPixels(this, 20);
		view.setPadding(p, p, p, p);
		
		// ...
	}	
}
```

Note for static values you could also work around this by simply defining your dimension value in a Android resource bucket and referencing it from there (then you get all the goodness that comes with Android resource buckets, such as assigning different values for different screen sizes). For cases where you need to calculate these values dynamically, having this functionality in a utility class can be quite handy

Happy coding everyone!