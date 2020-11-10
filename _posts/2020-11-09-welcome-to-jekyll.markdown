---
layout: post
title:  "A case for switch with errors!"
date:   2020-11-09 20:28:08 -0700
categories: go errors
---
I've found that when handling an error that requires multiple checks it's cleaner to use a switch than a if statement.

Lets use the example of a function that creates a directory if it doesn't exist.

In this example we use traditional if statements to do our error handling.
{% highlight go %}
func createIfNotExists(path string) error {
	stat, err := os.Stat(path)
	if err != nil {
		// if we got an error that this doesn't exist try creating it
		if os.IsNotExist(err) {
			err = os.MkdirAll(adrPath, 0755)
			if err != nil {
				return errors.Wrap(err, "couldn't create directory")
			}

			return nil
		}

		return errors.Wrap(err, "couldn't stat directory")
	}

	if !stat.IsDir() {
		return errors.New(adrPath + " exists and is not a directory")
	}

	return nil
}
{% endhighlight %}

In the above example the code is deeply nested and has various paths that can be confusing at first glance.

In the example below we use an empty switch and simply check for various scenarios we want to handle.
I also think this approach allows for a cleaner way to add more checks like our is directory check.
{% highlight go %}
func createIfNotExists(path string) error {
	stat, err := os.Stat(path)
	switch {
	// stat was successful something exists here verify its a directory
	case err == nil:
		if !stat.IsDir() {
			return errors.New(path + " exists and is not a directory")
		}

		return nil

	// is not exists error do nothing so we create the directory
	case os.IsNotExist(err):

	// some other error was encountered return it
	case err != nil:
		return errors.Wrap(err, "couldn't stat ADR directory")
	}

	return nil
}
{% endhighlight %}

This is of course all opinion based and I'm sure there are cases where the if statement is cleaner.
Let me know your thoughts which would you chose?

