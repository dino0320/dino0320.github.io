---
layout: my-post
title: "Inertia.js + shadcn/ui Form on Laravel"
date: 2025-10-15 00:00:00 +0000
categories: web-application-framework laravel
page_name: implement-inertia-and-shadcn-form-on-laravel-en
lang: en
image: /assets/images/web-application-framework/laravel/implement-inertia-and-shadcn-form-on-laravel-en/image3.png
---

This article explains how to implement an Inertia.js + shadcn/ui form in a Laravel project.

![Thumbnail](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image3.png "Thumbnail")

## References
- [shadcn/ui Form](https://ui.shadcn.com/docs/components/form)
- [inertia.js Pages](https://inertiajs.com/pages)

## Prerequisite
- You already have a Laravel project set up with React.  
For more information, refer to [this article](/web-application-framework/laravel/set-up-laravel-with-react-project-en).

## Environment
- Windows 11
- Ubuntu 24.04.3 LTS (WSL2 distribution)
- Docker Engine 28.4.0
- Amazon Linux 2023(OS of the Docker container)
- PHP 8.2.15 (fpm-fcgi)
- NGINX 1.24.0
- Laravel 12
- Node.js 20.14.0
- Vite 7.1.9
- React 19.2.0
- @inertiajs/react 2.2.6

## Implementation Steps
1. [Install shadcn/ui Form](#1-install-shadcnui-form)
2. [Implement a Test Form](#2-implement-a-test-form)

## 1. Install shadcn/ui Form
Run the following command to install the shadcn/ui Form component:

```bash
npx shadcn@latest add form
```

After running this command, you should see a `form.tsx` file in the `resources/js/components/ui` directory.

## 2. Implement a Test Form
Let's create a test form using Inertia.js and shadcn/ui Form.

Create a file named `resources/js/Pages/TestForm.tsx` based on the [shadcn/ui documentation](https://ui.shadcn.com/docs/components/form).

`TestForm.tsx`:

```tsx
import { zodResolver } from "@hookform/resolvers/zod"
import { useForm } from "react-hook-form"
import { z } from "zod"
 
import { Button } from "@/components/ui/button"
import {
  Form,
  FormControl,
  FormDescription,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form"
import { Input } from "@/components/ui/input"
 
const formSchema = z.object({
  username: z.string().min(2, {
    message: "Username must be at least 2 characters.",
  }),
})
 
export default function TestForm() {
  // 1. Define your form.
  const form = useForm<z.infer<typeof formSchema>>({
    resolver: zodResolver(formSchema),
    defaultValues: {
      username: "",
    },
  })
 
  // 2. Define a submit handler.
  function onSubmit(values: z.infer<typeof formSchema>) {
    // Do something with the form values.
    // ✅ This will be type-safe and validated.
    console.log(values)
  }
 
  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-8">
        <FormField
          control={form.control}
          name="username"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Username</FormLabel>
              <FormControl>
                <Input placeholder="shadcn" {...field} />
              </FormControl>
              <FormDescription>
                This is your public display name.
              </FormDescription>
              <FormMessage />
            </FormItem>
          )}
        />
        <Button type="submit">Submit</Button>
      </form>
    </Form>
  )
}

```

Install any missing packages.

Next, modify the `onSubmit` function to post data to Laravel via Inertia:

```tsx
import { router } from '@inertiajs/react' // Added

// ...

  // 2. Define a submit handler.
  function onSubmit(values: z.infer<typeof formSchema>) {
    // Do something with the form values.
    // ✅ This will be type-safe and validated.
    console.log(values)
    // Post values to /show-username
    router.post('/show-username', values) // Added
  }

// ...

```

Generate a controller using Artisan:

```bash
php artisan make:controller TestFormController
```

Then, edit `app/Http/Controllers/TestFormController.php` as follows.  
The `showUsername` method receives the `username` value from the form and renders the `Welcome` page with that data.

`TestFormController.php`:

```php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Inertia\Inertia;
use Inertia\Response;

class TestFormController extends Controller
{
    /**
     * Show username
     *
     * @param Request $request
     * @return Response
     */
    public function showUsername(Request $request): Response
    {
        $request->validate([
            'username' => 'required|string|max:255',
        ]);

        return Inertia::render('Welcome', [
          'username' => $request->username,
        ]);
    }
}

```

Create a `resources/js/Pages/Welcome.tsx` file referring to the [Inertia.js documentation](https://inertiajs.com/pages).

`Welcome.tsx`:

```tsx
import { Head } from '@inertiajs/react'

export default function Welcome({ username }) {
  return (
    <div>
      <Head title="Welcome" />
      <h1>Welcome</h1>
      <p>Hello {username}, welcome to your first Inertia app!</p>
    </div>
  )
}

```

Finally, edit `routes/web.php` as follows:

`web.php`:

```php
<?php

use App\Http\Controllers\TestFormController;
use Illuminate\Support\Facades\Route;
use Inertia\Inertia;

Route::get('/test-form', function () {
    return Inertia::render('TestForm');
});

Route::post('/show-username', [TestFormController::class, 'showUsername']);

```

Now open the `/test-form` page in your browser.  
You should see a form like this:

![Test form](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image1.png "Test form")

Enter any username and click the "Submit" button.
You should be redirected to the "Welcome" page successfully.

![Welcome page](/assets/images/{{ page.categories[0] }}/{{ page.categories[1] }}/{{ page.page_name }}/image2.png "Welcome page")