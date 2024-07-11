<script lang="ts">
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import Bottomtext from "$lib/components/Footer/Bottomtext.svelte";
    import countries from '$lib/countries';
    import { Input } from '@sveltestrap/sveltestrap';
    import Modal from '$lib/components/Modal.svelte';
    import { formatBase64Image } from '$lib/stores';

    interface User {
        id: string;
        username: string;
        email: string;
        profilePicture: string;
        country: string;
        createdAt: string;
    }

    let user: User = {
        id: '',
        username: '',
        email: '',
        profilePicture: '',
        country: '',
        createdAt: ''
    };

    let country = "";
    let isEditing: boolean = false;
    let showModal: boolean = false;
    const userId = localStorage.getItem("userId");
    console.log(userId);

    async function fetchProfile(): Promise<void> {
        try {
            const response = await fetch(`https://webgroove-82906d5c43b2.herokuapp.com/api/user/profile/${userId}`, {
                method: "GET",
            });
            if (response.ok) {
                const data: User = await response.json();
                user = data;
                user.profilePicture = formatBase64Image(data.profilePicture);
            } else {
                console.error('Fehler beim Abrufen des Profils:', response.statusText);
            }
        } catch (error) {
            console.error('Fehler beim Abrufen des Profils:', error);
        }
    }

    onMount((): void => {
        fetchProfile();
    });

    function goBack(): void {
        goto('/');
    }

    function editProfile(): void {
        isEditing = true;
    }

    async function saveProfileChanges(): Promise<void> {
        try {
            const response = await fetch(`https://webgroove-82906d5c43b2.herokuapp.com/api/manageUser/${userId}`, {
                method: "PUT", // Use PUT or POST depending on your API design
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify(user)
            });
            if (response.ok) {
                const updatedUser: User = await response.json();
                user = updatedUser;
                isEditing = false; // Exit editing mode
            } else {
                console.error('Fehler beim Speichern des Profils:', response.statusText);
            }
        } catch (error) {
            console.error('Fehler beim Speichern des Profils:', error);
        }
    }

    function cancelEdit(): void {
        isEditing = false;
    }

    function confirmDelete(): void {
        showModal = true;
    }

    async function deleteProfile(): Promise<void> {
        showModal = false;
        try {
            const response = await fetch(`https://webgroove-82906d5c43b2.herokuapp.com/api/manageUser/${userId}`, {
                method: "DELETE",
            });
            if (response.ok) {
                localStorage.removeItem("userId");
                goto('/');
            } else if (response.status === 404) {
                alert('Benutzer nicht gefunden.');
            } else {
                console.error('Fehler beim Löschen des Profils:', response.statusText);
            }
        } catch (error) {
            console.error('Fehler beim Löschen des Profils:', error);
        }
    }

    function cancelDelete(): void {
        showModal = false;
    }


</script>

<style>
    /* Add your styles here */
    .profile {
        margin: 4rem auto;
        max-width: 600px;
        padding: 2rem;
        background: #D9D9D9;
        border-radius: 8px;
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    }
    .profile h1 {
        margin-bottom: 1rem;
        color: black;
    }
    .profile p {
        display: flex;
        justify-content: space-between;
        margin: 0.5rem 0;
        color: black;
    }
    
    .profile p strong {
        flex: 1;
        text-align: left;
    }
    .profile p span {
        flex: 1;
        text-align: left; 
    }
    .back-button {
        display: inline-block;
        margin-top: 2rem;
        padding: 0.5rem 1rem;
        background: #007bff;
        color: white;
        text-decoration: none;
        border-radius: 4px;
        cursor: pointer;
        border: none;
    }
    .back-button:hover {
        background: #0056b3;
        transform: scale(1.1);
    }
    .edit-button {
        align-self: flex-end;
        display: inline-block;
        padding: 0.5rem 1rem;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
    .edit-button:hover {
        background: #0056b3;
        transform: scale(1.1);
    }
    .save-button, .cancel-button {
        display: inline-block;
        padding: 0.5rem 1rem;
        background: #007bff;
        color: white;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
    .save-button:hover {
        background: #0056b3;
        transform: scale(1.1);
    }
    .cancel-button {
        background: #dc3545;
    }
    .cancel-button:hover {
        background: #c82333;
        transform: scale(1.1);
    }
    .delete-button {
        background: #dc3545;
        color: white;
        display: inline-block;
        padding: 0.5rem 1rem;
        border: none;
        border-radius: 4px;
        cursor: pointer;
    }
    .delete-button:hover {
        background: #c82333;
        transform: scale(1.1);
    }
    
    .input-field {
        margin-bottom: 1rem;
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .input-field label {
        margin-bottom: 0.5rem;
        color: black; /* Set the label text color to black */
    }
    .input-field input {
        width: 75%; /* Make the input fields 25% smaller */
        padding: 0.5rem;
        border: 1px solid #ccc;
        border-radius: 4px;
        text-align: center; /* Center text in the input field */
    }
    .button-container {
        display: flex;
        justify-content: space-between;
        margin-top: 1rem;
    }
    .form-group {
        display: flex;
        flex-direction: column;
        margin-bottom: 15px;
        margin-left: 4em; 
        margin-right: 4em; 
        gap: 2px
    }
    .form-group label {
        color: black; /* Set label text color to black */
        text-align: center; /* Center the text */
    }

    .profile-picture {
        max-width: 200px; /* Maximale Breite des Profilbildes */
        border-radius: 50%; /* Rundes Profilbild */
        margin: 1rem auto; /* Zentriert das Profilbild */
        box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1); /* Leichte Schatten */
    }
    
</style>

<div class="profile">
    <h1>Profil</h1>
    {#if isEditing}
        <div class="input-field">
            <label for="username">Username:</label>
            <input type="text" id="username" bind:value={user.username} />
        </div>
        <div class="input-field">
            <label for="email">Email:</label>
            <input type="email" id="email" bind:value={user.email} />
        </div>
        <div class="form-group">
            <label for="country">Land</label>
            <Input name="country" type="select" bind:value={user.country}>
                {#each countries as country}
                    <option value={country.value}>{country.label}</option>
                {/each}
            </Input>
        </div>
        <div class="button-container">
            <button class="save-button" on:click={saveProfileChanges}>Speichern</button>
            <button class="cancel-button" on:click={cancelEdit}>Abbrechen</button>
            <button class="delete-button" on:click={confirmDelete}>Profil löschen</button>
        </div>
    {:else}
        <img src={user.profilePicture} alt="Profilbild" class="profile-picture">
        <p><strong>Username:</strong> <span>{user.username}</span></p>
        <p><strong>Email:</strong> <span>{user.email}</span></p>
        <p><strong>Country:</strong> <span>{user.country ? user.country : 'kein Land angegeben'}</span></p>
        <p><strong>Erstellt am:</strong> <span>{new Date(user.createdAt).toLocaleDateString()}</span></p>
        <div style="display: flex; justify-content: space-between;">
            <button class="back-button" on:click={goBack}>Zurück</button>
            <button class="edit-button" on:click={editProfile}>Bearbeiten</button>
        </div>
    {/if}
</div>

<Modal
    show={showModal}
    message="Bist du sicher, dass du dein Profil löschen möchtest? Diese Aktion kann nicht rückgängig gemacht werden."
    onConfirm={deleteProfile}
    onCancel={cancelDelete}
/>

<Bottomtext/>
