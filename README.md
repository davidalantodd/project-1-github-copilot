# Project 1: Refactoring Legacy Code for Scalability and Maintainability

## Task 1: Review and Analyze the Legacy Codebase

### Prompt
@workspace Identify code blocks, looking for any code that is inefficient. Look for overly complex or redundant code, such as large monolithic functions or repeated logic, and note unclear or poorly documented sections. Also, look out for any performance bottlenecks, or areas that might slow execution, such as inefficient loops or poorly optimized database queries. Give me three examples that meet these criteria and show me the locations of where these code blocks exist, including the file location and line number.

### Output:
Here are three examples of inefficient or redundant code blocks in your workspace:

#### 1. Redundant State Update in FavoritesProvider:

- Location: FavoritesProvider.js, lines 10-18
- Issue: The addFavorite and removeFavorite functions are creating new arrays unnecessarily, which can be optimized.
- Code:
    ```javascript
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
    ```javascript
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
    ```javascript
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
    ```javascript
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
    ```javascript
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
    ```javascript
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
With this information, I plan to tackle these three key issues:
1. Redundant Fetch Call in PokemonCard
    - We can optimize the performance of this application by limiting the amount of times the fetch call is made by caching the results, that way if the data has already been fetched once, we can reuse that stored data instead of re-calling fetch.
    - I think this will have the biggest impact on performance, since these calls are happening in realtime
    - Code:
        ```javascript
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
        ```javascript
        useEffect(() => {
        setFilteredPokemon(
            pokemonList.filter((pokemon) =>
            pokemon.name.toLowerCase().includes(search.toLowerCase())
            )
        );
        }, [search, pokemonList]);
        ```
3. Redundant State Update in FavoritesProvider
    - We can optimize the state setters by changing the way we are mutating the favorites array, since we don't need to create new arrays each time.
    - This will have a slight impact on performance, but to users, it might not be as noticeable since it is estimated to be just milliseconds of improvement
    - Code:
    ```javascript
    function addFavorite(favorite) {
    setFavorites([...favorites, favorite]);
    }

    function removeFavorite(name) {
    setFavorites(favorites.filter((favorite) => name !== favorite));
    }
    ```

Other Notes:
- I tried this on a few different repos, for some public GitHub repos, GitHub Copilot wouldn't generate code that is publicly available due to the settings of our license (Since we have the setting ‘Enabling or disabling suggestions matching public code,’). I got around that by prompting Copilot to only give me file locations/line numbers instead of generating the code, since it would otherwise block those outputs from being generated
- When clicking on links in outputs, sometimes it would bring me to the wrong file (e.g. clicking on the link for PokemonCard.js brought me to PokemonCard.test.js)

## Task 2: Refactor the Code


## Task 3: Leverage GitHub Copilot


## Task 4: Write and Execute Unit Tests


