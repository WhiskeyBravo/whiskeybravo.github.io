---
layout: post
date: 2015-10-01 15:52
title: Constructing Android Fragments The Right Way™
published: true
tags: android, java, fragment
---

Eager to use [Fragments][1] in your Android apps?  Take a minute and learn how to properly implement constructors for the Android [Fragment][1] pattern and avoid a common pitfall.

###The Problem
As one might guess, you might pass arguments to a new [Fragment][1] by modifying the constructor.  This is in fact **not** the correct way to implement a [Fragment][1] in the Android style.

```java
public class MyAwesomeFragment extends Fragment {
    private int argOne;
    private int argTwo;
    
    public MyAwesomeFragment(int memberOne, int memberTwo) {
      this.argOne = memberOne;
      this.argTwo = memberTwo;
    }
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // SETUP CODE HERE
    }
    
     @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_projected_run_details, container, false);
        // Pull out view members for later use....
        return view;
}
```

###The Fix
The correct way, [as stated by the documentation][2], is to set and pull your arguments out of Bundle objects.  This is important because Android uses this technique to destroy and regenerate your fragments as needed.  If your fragment is destroyed and you don't use this approach for arguments, your application may crash or be restored in an inconsistent state.

> Every fragment must have an empty constructor, so it can be instantiated when restoring its activity's state.
    

```java
public class MyAwesomeFragment extends Fragment {
    private int argOne;
    private int argTwo;
    
    // Argument identifiers
    private static final String ARG_ONE = "arg_one";
    private static final String ARG_TWO = "arg_two";
    
    // fragments are required to have empty constructors
    public MyAwesomeFragment() {}
    
    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        // pull out your arguments here!
        if( getArguments() != null ) {
            this.argOne = getArguments().getInt(ARG_ONE);
            this.argTwo = getArguments().getInt(ARG_TWO);
        }
        // SETUP CODE HERE
    }
    
     @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        super.onCreateView(inflater, container, savedInstanceState);
        // Inflate the layout for this fragment
        View view = inflater.inflate(R.layout.fragment_projected_run_details, container, false);
        // Pull out view members for later use....
        return view;
}
```

Now rest easy knowing the Android Fragment management subsystem will be happy with your custom Fragments.



[1]: <http://developer.android.com/reference/android/app/Fragment.html>
[2]: <http://developer.android.com/reference/android/app/Fragment.html#Fragment()>
