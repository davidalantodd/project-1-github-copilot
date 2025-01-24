# Project 1: Refactoring Legacy Code for Scalability and Maintainability

Link to Source Code: [Pokeverse Solution Optimized with GitHub Copilot](https://github.com/davidalantodd/pokeverse-solution-github-copilot/tree/week-4-github-copilot-refactor)

## Task 1: Review and Analyze the Legacy Codebase

### Prompt
@workspace Identify code blocks, looking for any code that is inefficient. Look for overly complex or redundant code, such as large monolithic functions or repeated logic, and note unclear or poorly documented sections. Also, look out for any performance bottlenecks, or areas that might slow execution, such as inefficient loops or poorly optimized database queries. Give me three examples that meet these criteria and show me the locations of where these code blocks exist, including the file location and line number.

### Output:
Here are three examples of inefficient or redundant code blocks in your workspace:

#### 1. Redundant State Update in FavoritesProvider:

- Location: FavoritesProvider.js, lines 10-18
- Issue: The addFavorite and removeFavorite functions are creating new arrays unnecessarily, which can be optimized.
- Code:
    ```js
    function addFavorite(favorite) {
    setFavorites([...favorites, favorite]);
    }

    function removeFavorite(name) {
    setFavorites(favorites.filter((favorite) => name !== favorite));
    }
    ```

#### 2. Redundant Fetch Call in PokemonCard

- Location: PokemonCard.js, lines 10-16
- Issue: The fetch call in the useEffect hook is made every time the url changes, which might be redundant if the data is already available.
- Code:
    ```js
    useEffect(() => {
    fetch(url)
        .then((res) => res.json())
        .then((data) => {
        setPokemon(data);
        })
        .catch((error) => {
        console.error(error);
        });
    }, [url]);
    ```

#### 3. Inefficient Filtering in App Component

- Location: App.js, lines 22-28
- Issue: The setFilteredPokemon function is called on every render, which can be optimized by debouncing the search input.
- Code:
    ```js
    useEffect(() => {
    setFilteredPokemon(
        pokemonList.filter((pokemon) =>
        pokemon.name.toLowerCase().includes(search.toLowerCase())
        )
    );
    }, [search, pokemonList]);
    ```
These code blocks can be optimized to improve performance and reduce redundancy.

### Prompt:
I wanted a few more options so I prompted Copilot with "Can you generate two more inefficiencies?". If gave me two more to choose from:

### Output:
#### 4. Inefficient State Management in PokemonList Component

- Location: src/components/PokemonList.js, lines 15-23
- Issue: The state is being updated multiple times within a single function, which can cause unnecessary re-renders.
- Code:
    ```js
    const fetchPokemon = async () => {
    const response = await fetch('https://pokeapi.co/api/v2/pokemon');
    const data = await response.json();
    setPokemonList(data.results);
    setLoading(false);
    };

    useEffect(() => {
    fetchPokemon();
    }, []);
    ```

#### 5. Unoptimized Loop in PokemonDetails Component

- Location: src/components/PokemonDetails.js, lines 30-38
- Issue: The loop is iterating over the entire list of Pokémon abilities even if the desired ability is found early in the list.
- Code:
    ```js
    const getAbilityDetails = async () => {
    for (let i = 0; i < abilities.length; i++) {
        const response = await fetch(abilities[i].ability.url);
        const data = await response.json();
        if (data.name === desiredAbility) {
        setAbilityDetails(data);
        break;
        }
    }
    };

    useEffect(() => {
    getAbilityDetails();
    }, [abilities]);
    ```

### Prompting further:
- Suggestions 1-3 are solid. Suggestions 4 and 5 are actually located on a different branch of the repo, so I did some prompting to ask Copilot to "generate two more that are on the current branch week-4 and to only look at files that are only readily available in my editor"
- After that prompting, it generated two suggestions that weren't on any of the current branches. I noticed since the links to the files weren't working and actually the filename didn't exist anywhere. So my assumption is that it was code in a previous commit or a complete hallucination since I couldn't find it in any of the branches.
- For this reason, it's important to always verify that these files and the generated code exists before taking action. More refined prompting could also help with more accurate results.

Example of code that wasn't present in latest commits on on all branches (SearchBar.js component doesn't exist):

#### Inefficient Event Handling in SearchBar:

- Location: src/components/SearchBar.js, lines 10-18
- Issue: The state is being updated on every keystroke, which can be optimized by debouncing the input.
- Code:
    ```js
    const handleChange = (event) => {
    setSearch(event.target.value);
    };

    return (
    <input
        type="text"
        value={search}
        onChange={handleChange}
        placeholder="Search Pokémon"
    />
    );
    ```

### Key Issues
With this information, I plan to tackle these two key issues:
1. Redundant Fetch Call in PokemonCard
    - We can optimize the performance of this application by limiting the amount of times the fetch call is made by caching the results, that way if the data has already been fetched once, we can reuse that stored data instead of re-calling fetch.
    - I think this will have the biggest impact on performance, since these calls are happening in realtime
    - Code:
        ```js
        useEffect(() => {
        fetch(url)
            .then((res) => res.json())
            .then((data) => {
            setPokemon(data);
            })
            .catch((error) => {
            console.error(error);
            });
        }, [url]);
        ```
2. Inefficient Filtering in App Component
    - We can optimize the amount of times we call setFilteredPokemon, so instead of rerendering every time the user types a letter, we can debounce the search input so it only rerenders after the user stops typing their query.
    - This should have a noticeable impact on user experience, but it might come down to preference from the Product team if this optimization is even wanted.
    - Code:
        ```js
        useEffect(() => {
        setFilteredPokemon(
            pokemonList.filter((pokemon) =>
            pokemon.name.toLowerCase().includes(search.toLowerCase())
            )
        );
        }, [search, pokemonList]);
        ```

I chose not to move forward with "Redundant State Update in FavoritesProvider." We could have optimized the state setters by changing the way we are mutating the favorites array, since we don't need to create new arrays each time. This would have had a slight impact on performance, but to users, it might not be as noticeable since it is estimated to be just milliseconds of improvement. After using Copilot and doing some more research, I feel confident moving forward with the other two refactors.


Other Notes:
- I tried this on a few different repos, for some public GitHub repos, GitHub Copilot wouldn't generate code that is publicly available due to the settings of our license (Since we have the setting ‘Enabling or disabling suggestions matching public code,’). I got around that by prompting Copilot to only give me file locations/line numbers instead of generating the code, since it would otherwise block those outputs from being generated
- When clicking on links in outputs, sometimes it would bring me to the wrong file (e.g. clicking on the link for PokemonCard.js brought me to PokemonCard.test.js)

## Task 2: Refactor the Code & Task 3: Leverage GitHub Copilot

### 1. Redundant Fetch Call in PokemonCard
We can optimize the performance of this application by limiting the amount of times the fetch call is made by caching the results, that way if the data has already been fetched once, we can reuse that stored data instead of re-calling fetch.

    ```js
        function PokemonCard({ url, name }) {
            const [pokemon, setPokemon] = useState(null);
            const { favorites, addFavorite, removeFavorite } = useContext(FavoritesContext);

            useEffect(() => {
                fetch(url)  // this fetch is called every time the url of the pokemon changes, even if the same pokemon is called multiple times
                .then((res) => res.json())
                .then((data) => {
                    setPokemon(data);
                })
                .catch((error) => {
                    console.error(error);
                });
            }, [url]);

            return (
                <Card>
                    {/* existing code */}
                </Card>
            )
        }
    ```
#### Prompt: 
I asked Copilot to generate a solution in more detail by asking "Can you explain this a bit further and talk about how I might implement a fix? 'Redundant Fetch Call in PokemonCard'"

#### Output:

Possible Fix:
- Implement Caching: Use a simple caching mechanism to store the fetched data. If the data for a particular URL is already cached, use the cached data instead of making a new API call.

Implementation:
- Create a cache object to store the fetched data.
- Check if the data for the given URL is already in the cache.
- If the data is in the cache, use it directly.
- If the data is not in the cache, fetch it from the API and store it in the cache.

Here is the code suggestion Copilot produced with some comments I added in:

    ```js
    // existing code

    const cache = {};   // implementing cache to store individual pokemon data

    function PokemonCard({ url, name }) {
        const [pokemon, setPokemon] = useState(null);
        const { favorites, addFavorite, removeFavorite } = useContext(FavoritesContext);

        useEffect(() => {
            if (cache[url]) {   // checking for cached result
                setPokemon(cache[url]);
            } else {    // if it hasn't been cached, fetch the data
            fetch(url)
                .then((res) => res.json())
                .then((data) => {
                cache[url] = data; // storing result in the cache
                setPokemon(data);
                })
                .catch((error) => {
                console.error(error);
                });
            }
        }, [url]);
        // existing code
    }
    ```

    This approach looked good, so I went ahead with the implementation. Validation and testing in next section.


2. Inefficient Filtering in App Component
We can optimize the amount of times we call setFilteredPokemon, so instead of rerendering every time the user types a letter, we can debounce the search input so it only rerenders after the user stops typing their query.

    ```js
    useEffect(() => {
        setFilteredPokemon(
            pokemonList.filter((pokemon) =>
            pokemon.name.toLowerCase().includes(search.toLowerCase())
            )
        );
    }, [search, pokemonList]);

    return (
        <div data-testid="app">
            {/* existing code */}
                <InputGroup>
                    <InputGroup.Text id='search'>Search</InputGroup.Text>
                        <FormControl
                        value={search}
                        aria-label='search'
                        aria-describedby='search'
                        onChange={(e) => setSearch(e.target.value)}
                    />
                </InputGroup>
            { /* existing code */}
        </div>
    )
    ```

#### Prompt: 
I again asked Copilot to expand on this solution.

#### Output:

Possible Fix:
- Debounce the Search Input: Use a debounce function to limit the rate at which the search state is updated. This will reduce the number of times the filtering function is called.

Implementation:
- Use the lodash.debounce library to debounce the search input.
- Update the handleChange function to use the debounced function.
- Update the useEffect hook to filter the list based on the debounced search input.
- Here's how you can implement this:

Here is the code suggestion Copilot produced with some comments I added in:

    ```js
    import React, { useState, useEffect, useCallback } from 'react';
    import debounce from 'lodash.debounce'; // import debounce library

    function App() {
    const [search, setSearch] = useState('');
    const [filteredPokemon, setFilteredPokemon] = useState([]);
    const [pokemonList, setPokemonList] = useState([]);

    // Debounce the search input to limit the rate of state updates
    const debouncedSetSearch = useCallback(
        debounce((value) => setSearch(value), 300),
        []
    );

    useEffect(() => {
        setFilteredPokemon(
        pokemonList.filter((pokemon) =>
            pokemon.name.toLowerCase().includes(search.toLowerCase())
        )
        );
    }, [search, pokemonList]);

    const handleChange = (event) => {
        debouncedSetSearch(event.target.value); //call the new debouncedSetSearch function
    };

    return (
        <div>
        <input
            type="text"
            value={search}
            onChange={handleChange} // hange this onChange to call the new function
            placeholder="Search Pokémon"
        />
        <div>
            {filteredPokemon.map((pokemon) => (
            <div key={pokemon.name}>{pokemon.name}</div>
            ))}
        </div>
        </div>
    );
    }

    export default App;
    ```

    This approach seemed promising, so I went ahead with the implementation. When implementing, this time I used the "Apply in Editor" function next to the generated code to automatically insert into the file. This worked well and I could accept individual code changes. Validation and testing in next section.

Screenshot of accepting Copilot's code changes:

![Image](/screenshots/code-changes.png)

## Task 4: Write and Execute Unit Tests

### 1. Redundant Fetch Call in PokemonCard
The implementation here continued to pass existing tests. It also functioned correctly by running the application and completing a few actions via the UI. I also created an additional unit test to test if there was a performance difference by rendering the same Pokemon Card component 100 times. Here is the test:

```js
it('Should render multiple times', async () => {
    for (let i = 0; i < 100; i++){
      await act(async () => {
        render(
          <FavoritesProvider>
              <PokemonCard name={MOCK_ONE_POKEMON_DATA.species.name} url={MOCK_ONE_POKEMON_DATA.species.url} />
          </FavoritesProvider>
        );
      });
    }
});
```

#### Test results
Before caching (test took 160ms)

![Image](/screenshots/test-1-before.png)

After caching (test took 88ms)

![Image](/screenshots/test-1-after.png)

#### 2. Inefficient Filtering in App Component
After testing out the initial change via performing actions on the UI, it actually made the user experience worse. As a user, after typing one character in the search bar, I needed to wait until the debouncing time limit ended (.3 ms) before typing another letter in the search bar.

So, I prompted Copilot again with "I implemented this and now when I type a character, it has to execute before I can type another, is there any way I can type as many characters as I want but it updates in regular intervals?"

It suggested to add another state called "input" that would track the search bar contents, but still only update the search state at regular intervals. This worked a lot better in testing. It was still a little too slow to respond, so I changed the value for updating to be every .2 seconds rather than .3 and that felt more user-responsive, while still limiting the number of times the filter needs to be executed.

I also adjusted the test to be reflective of the new lag in updating the filtered Pokemon. Here's the new test:

```js
it('should search and filter', async () => {
    await act(async () => {
      render(<App />);
    });
    const input = screen.getByRole('textbox');
    expect(input.value).toBe('');
    fireEvent.change(input, {target: {value: 'ch'}})
    expect(input.value).toBe('ch');
    await waitFor(() => {
      expect(screen.getAllByTestId('PokemonCard').length).toEqual(11);
    })
  })
```

#### Test results

![Image](/screenshots/test-2.png)

Although the test takes longer, the user experience is improved, and the new application is scalable, so if the list of Pokemon grew a ton, the filter action wouldn't slow down users from typing.

Overall, testing allowed us to confirm functionality of the existing code and test some performance optimizations on the refactored code.

## Task 5: Document the Refactored Code

Link to Source Code: [Pokeverse Solution Optimized with GitHub Copilot](https://github.com/davidalantodd/pokeverse-solution-github-copilot/tree/week-4-github-copilot-refactor)
