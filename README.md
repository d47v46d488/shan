# shan// index.js
import React, { useEffect, useState } from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';
import { configureStore, createSlice, createAsyncThunk } from '@reduxjs/toolkit';
import axios from 'axios';
import { List } from 'antd';

const pokemonSlice = createSlice({
  name: 'pokemon',
  initialState: { data: [], status: 'idle' },
  reducers: {},
  extraReducers: (builder) => {
    builder
      .addCase(fetchPokemon.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.data = action.payload;
      })
      .addCase(fetchPokemon.pending, (state) => {
        state.status = 'loading';
      });
  },
});

const fetchPokemon = createAsyncThunk('pokemon/fetchPokemon', async () => {
  const response = await axios.get('https://pokeapi.co/api/v2/pokemon/');
  return response.data.results;
});

const store = configureStore({
  reducer: { pokemon: pokemonSlice.reducer },
});

const PokemonApp = () => {
  const { status, data } = useSelector((state) => state.pokemon);
  const dispatch = useDispatch();

  useEffect(() => {
    dispatch(fetchPokemon());
  }, [dispatch]);

  return (
    <div>
      {status === 'loading' && <p>Loading...</p>}
      {status === 'succeeded' && (
        <List
          dataSource={data}
          renderItem={(pokemon) => (
            <List.Item>
              <img
                src={`https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/${pokemon.url.split('/')[6]}.png`}
                alt={pokemon.name}
              />
              {pokemon.name}
            </List.Item>
          )}
        />
      )}
    </div>
  );
};

render(
  <Provider store={store}>
    <PokemonApp />
  </Provider>,
  document.getElementById('root')
);
