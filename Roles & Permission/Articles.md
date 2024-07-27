
### Create Blade File Articles:
Let’s Create & Open the `resourch/views/articles/articles.blade.php`
```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                {{ __('Articles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('articles.create') }}">Create</a>
            </h2>
        </div>
    </x-slot>

  
    @if (Session::has('success'))
        <div class="bg-green-200 border-green-600 p-4 mt-3 mx-16 rounded-sm shadow-sm">{{ Session::get('success') }}</div>
    @endif

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <div class="relative overflow-x-auto">
                        <table class="w-full text-sm text-left rtl:text-right text-gray-500 ">
                            <thead class="text-sm font-bold text-gray-800 border-b border-gray-200 ">
                                <tr>
                                    <th scope="col" class="px-6 py-3 w-[25%]">
                                        Title
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[20%]">
                                        Author
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[30%]">
                                        Articles
                                    </th>
                                    <th scope="col" class="px-6 py-3 w-[25%]">
                                        Action
                                    </th>
                                </tr>
                            </thead>
                            <tbody>
                                @foreach ($articles as $article)
                                <tr class="bg-white border-b border-gray-200">
                                    <td class="px-6 py-4">
                                        {{ $article->title }}
                                    </td>
                                    <td class="px-6 py-4">
                                        {{ $article->author }}
                                    </td>
                                    <td class="px-6 py-4">
                                        {{ Str::limit($article->article, 100) }}
                                    </td>
                                    <td class="px-6 py-4">
                                        @can('edit articles')  
                                        <a class="bg-slate-700 text-white px-3 py-2 rounded-md" href="{{ route('articles.edit', $article->id) }}">Edit</a>
                                        @endcan

                                        @can('delete articles')
                                        <a class="bg-red-700 text-white px-3 py-2 rounded-md" href="{{ route('articles.destroy', $article->id) }}">Delete</a>
                                        @endcan
                                    </td>
                                </tr>
                                @endforeach
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/articles/articles_create.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Create Articles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('articles') }}">Back</a>
            </h2>
        </div>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <form action="{{ route('articles.store') }}" method="POST">
                        @csrf
                        <div>
                            <div>
                                <label for="" class="text-lg font-medium">Title</label>
                                <div class="mb-3">
                                    <input type="text" name="title" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter title" value="{{ old('title') }}">
                                </div>
                                @error('title')
                                <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>

                            <div>
                                <label for="" class="text-lg font-medium">Author</label>
                                <div class="mb-3">
                                    <input type="text" name="author" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter author name" value="{{ old('author') }}">
                                </div>
                                @error('author')
                                <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>
                           
                            <div>
                                <label for="" class="text-lg font-medium">Article</label>
                                <div class="mb-3">
                                    <textarea name="article" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter article">{{old('article')}}</textarea></textarea>
                                </div>
                                @error('article')
                               <div class=" text-red-600 mb-3">{{ $message }}</div>
                                @enderror
                            </div>
                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```

Let’s Create & Open the `resourch/views/articles/articles_edit.blade.php`

```html
<x-app-layout>
    <x-slot name="header">
        <div class="flex justify-between">
            <h2 class="font-semibold text-xl text-gray-800 leading-tight px-10">
                {{ __('Edit Articles') }}
            </h2>
            <h2 class="font-semibold text-lg text-gray-800 leading-tight px-10">
                <a href="{{ route('articles') }}">Back</a>
            </h2>

        </div>

    </x-slot>

  

    <div class="py-12">

        <div class="max-w-7xl mx-auto sm:px-6 lg:px-16">

            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">

                <div class="p-6 text-gray-900">

                    <form action="{{ route('articles.update', $articles->id) }}" method="POST">

                        @csrf

                        @method('PUT')

                        <div>

                            <div>

                                <label for="" class="text-lg font-medium">Title</label>

                                <div class="mb-3">

                                    <input type="text" name="title" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter title" value="{{ old('title', $articles->title) }}">

                                </div>

                                @error('title')

                                <div class=" text-red-600 mb-3">{{ $message }}</div>

                                @enderror

                            </div>

  

                            <div>

                                <label for="" class="text-lg font-medium">Author</label>

                                <div class="mb-3">

                                    <input type="text" name="author" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter author name" value="{{ old('author', $articles->author) }}">

                                </div>

                                @error('author')

                                <div class=" text-red-600 mb-3">{{ $message }}</div>

                                @enderror

                            </div>

  

                            <div>

                                <label for="" class="text-lg font-medium">Article</label>

                                <div class="mb-3">

                                    <textarea name="article" class="border-gray-300 shadow-sm w-1/2 rounded-md" placeholder="Enter article">{{old('article', $articles->article)}}</textarea></textarea>

                                </div>

                                @error('article')

                                <div class=" text-red-600 mb-3">{{ $message }}</div>

                                @enderror

                            </div>

  

                            <button type="submit" class="bg-slate-700 text-white px-3 py-2 rounded-md">Submit</button>

                        </div>

                    </form>

                </div>

            </div>

        </div>

    </div>

</x-app-layout>
```