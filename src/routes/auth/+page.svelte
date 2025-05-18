<script>
	import { toast } from 'svelte-sonner';

	import { onMount, getContext, tick } from 'svelte';
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';

	import { getBackendConfig } from '$lib/apis';
	import { ldapUserSignIn, getSessionUser, userSignIn, userSignUp } from '$lib/apis/auths';

	import { WEBUI_API_BASE_URL, WEBUI_BASE_URL } from '$lib/constants';
	import { WEBUI_NAME, config, user, socket } from '$lib/stores';

	import { generateInitialsImage, canvasPixelTest } from '$lib/utils';

	import Spinner from '$lib/components/common/Spinner.svelte';
	import OnBoarding from '$lib/components/OnBoarding.svelte';
	import { CheckIcon, Shield, ShieldCheck, X } from 'lucide-svelte';
	import Check from '$lib/components/icons/Check.svelte';

	const i18n = getContext('i18n');

	let loaded = false;

	let mode = $config?.features.enable_ldap ? 'ldap' : 'signin';

	let name = '';
	let email = '';
	let password = '';

	let ldapUsername = '';

	const querystringValue = (key) => {
		const querystring = window.location.search;
		const urlParams = new URLSearchParams(querystring);
		return urlParams.get(key);
	};

	const setSessionUser = async (sessionUser) => {
		if (sessionUser) {
			console.log(sessionUser);
			toast.success($i18n.t(`You're now logged in.`));
			if (sessionUser.token) {
				localStorage.token = sessionUser.token;
			}
			$socket.emit('user-join', { auth: { token: sessionUser.token } });
			await user.set(sessionUser);
			await config.set(await getBackendConfig());

			const redirectPath = querystringValue('redirect') || '/';
			goto(redirectPath);
		}
	};

	const signInHandler = async () => {
		const sessionUser = await userSignIn(email, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});

		await setSessionUser(sessionUser);
	};

	const signUpHandler = async () => {
		const sessionUser = await userSignUp(name, email, password, generateInitialsImage(name)).catch(
			(error) => {
				toast.error(`${error}`);
				return null;
			}
		);

		await setSessionUser(sessionUser);
	};

	const ldapSignInHandler = async () => {
		const sessionUser = await ldapUserSignIn(ldapUsername, password).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		await setSessionUser(sessionUser);
	};

	const submitHandler = async (event) => {
		event.preventDefault();

		if (mode === 'ldap') {
			await ldapSignInHandler();
		} else if (mode === 'signin') {
			await signInHandler();
		} else {
			await signUpHandler();
		}
	};

	const checkOauthCallback = async () => {
		if (!$page.url.hash) {
			return;
		}
		const hash = $page.url.hash.substring(1);
		if (!hash) {
			return;
		}
		const params = new URLSearchParams(hash);
		const token = params.get('token');
		if (!token) {
			return;
		}
		const sessionUser = await getSessionUser(token).catch((error) => {
			toast.error(`${error}`);
			return null;
		});
		if (!sessionUser) {
			return;
		}
		localStorage.token = token;
		await setSessionUser(sessionUser);
	};

	let onboarding = false;

	let triggerButton = null;

	let passwordStrength = 0;
	let passwordChecks = {
		minLength: false,
		hasUppercase: false,
		hasLowercase: false,
		hasNumber: false,
		hasSpecial: false
	};

	const validatePasswordStrength = (pass) => {
		passwordChecks = {
			minLength: pass.length >= 8,
			hasUppercase: /[A-Z]/.test(pass),
			hasLowercase: /[a-z]/.test(pass),
			hasNumber: /[0-9]/.test(pass),
			hasSpecial: /[^A-Za-z0-9]/.test(pass)
		};

		let strength = 0;

		if (passwordChecks.minLength) strength += 20;
		if (passwordChecks.hasUppercase) strength += 20;
		if (passwordChecks.hasLowercase) strength += 20;
		if (passwordChecks.hasNumber) strength += 20;
		if (passwordChecks.hasSpecial) strength += 20;

		passwordStrength = strength;

		return {
			strength,
			isValid: strength >= 60
		};
	};

	$: {
		if (password) {
			const { isValid } = validatePasswordStrength(password);

			const signupButton = document.querySelector('#signup-form button[type="submit"]');
			const signinButton = document.querySelector('#login-form button[type="submit"]');

			if (mode === 'signup' && signupButton) {
				signupButton.disabled = !isValid;
				signupButton.classList.toggle('opacity-50', !isValid);
				signupButton.classList.toggle('cursor-not-allowed', !isValid);
			} else if (mode === 'signin' && signinButton) {
				signinButton.disabled = false;
				signinButton.classList.remove('opacity-50', 'cursor-not-allowed');
			}
		}
	}

	async function setLogoImage() {
		await tick();
		const logo = document.getElementById('logo');

		if (logo) {
			const isDarkMode = document.documentElement.classList.contains('dark');

			if (isDarkMode) {
				const darkImage = new Image();
				darkImage.src = '/static/favicon-dark.png';

				darkImage.onload = () => {
					logo.src = '/static/favicon-dark.png';
					logo.style.filter = ''; // Ensure no inversion is applied if favicon-dark.png exists
				};

				darkImage.onerror = () => {
					logo.style.filter = 'invert(1)'; // Invert image if favicon-dark.png is missing
				};
			}
		}
	}

	onMount(async () => {
		if ($user !== undefined) {
			const redirectPath = querystringValue('redirect') || '/';
			goto(redirectPath);
		}
		await checkOauthCallback();

		loaded = true;
		setLogoImage();

		if (($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false) {
			await signInHandler();
		} else {
			onboarding = true;
		}

		if (triggerButton) {
			triggerButton.click();
		}
	});
</script>

<svelte:head>
	<title>
		{`${$WEBUI_NAME}`}
	</title>
</svelte:head>

<OnBoarding
	bind:show={onboarding}
	getStartedHandler={() => {
		onboarding = false;
		mode = 'signin';
	}}
/>

<div class="w-full h-screen max-h-[100dvh] text-white relative">
	<div class="w-full h-full absolute top-0 left-0 bg-white dark:bg-black"></div>

	<div class="w-full absolute top-0 left-0 right-0 h-8 drag-region" />
	{#if loaded}
		<div class="fixed m-10 z-50">
			<div class="flex space-x-2">
				<div class=" self-center">
					<img
						id="logo"
						crossorigin="anonymous"
						src="{WEBUI_BASE_URL}/static/splash.png"
						class=" w-6 rounded-full"
						alt=""
					/>
				</div>
			</div>
		</div>

		<div
			class="fixed bg-transparent min-h-screen w-full flex justify-center font-primary z-50 text-black dark:text-white"
		>
			<div class="w-full sm:max-w-sm px-10 min-h-screen flex flex-col text-center">
				{#if ($config?.features.auth_trusted_header ?? false) || $config?.features.auth === false}
					<div class=" my-auto pb-10 w-full">
						<div
							class="flex items-center justify-center gap-3 text-xl sm:text-2xl text-center font-semibold dark:text-gray-200"
						>
							<div>
								{$i18n.t('Signing in to {{WEBUI_NAME}}', { WEBUI_NAME: $WEBUI_NAME })}
							</div>

							<div>
								<Spinner />
							</div>
						</div>
					</div>
				{:else}
					<div class="  my-auto pb-10 w-full dark:text-gray-100">
						<!-- Полностью переработанная форма с фокусом на совместимость с функциями автозаполнения браузеров -->

						{#if mode === 'signin'}
							<!-- Форма входа - ВАЖНО: отдельная форма для логина -->
							<form
								id="login-form"
								name="login-form"
								method="post"
								autocomplete="on"
								on:submit={submitHandler}
								class="flex flex-col justify-center"
							>
								<div class="mb-1">
									<div class="text-2xl font-medium">
										{$i18n.t('Вход в аккаунт')}
									</div>
								</div>

								<div class="flex flex-col gap-3 mt-4">
									<!-- Email поле для входа -->
									<div class="relative">
										<input
											bind:value={email}
											type="email"
											id="login-email"
											name="username"
											class="peer block w-full rounded-2xl border py-3 px-4 text-neutral-900 focus:border-blue-500 dark:text-white shadow-xs outline-none border-neutral-200 dark-border-neutral-800 sm:text-sm sm:leading-6 transition-all"
											autocomplete="username"
											required
										/>
										<label
											for="login-email"
											class="absolute left-2 text-sm top-1/2 -translate-y-1/2 text-neutral-500 bg-white dark:bg-neutral-800 rounded-lg transition-all duration-200 peer-focus:text-neutral-800 peer-focus:top-0 peer-focus:left-2 px-2 peer-focus:text-[12px] {email
												? 'text-[12px] !top-0 !left-2'
												: ''}"
										>
											{$i18n.t('Email')}
										</label>
									</div>

									<!-- Поле пароля для входа -->
									<div class="relative">
										<input
											bind:value={password}
											type="password"
											id="login-password"
											name="password"
											class="peer block w-full rounded-2xl border py-3 px-4 pr-12 text-neutral-900 focus:border-blue-500 dark:text-white shadow-xs outline-none border-neutral-200 dark-border-neutral-800 sm:text-sm sm:leading-6 transition-all"
											autocomplete="current-password"
											required
										/>
										<label
											for="login-password"
											class="absolute left-2 text-sm top-1/2 -translate-y-1/2 text-neutral-500 bg-white dark:bg-neutral-800 rounded-lg transition-all duration-200 peer-focus:text-[12px] peer-focus:text-neutral-800 peer-focus:top-0 peer-focus:left-2 px-2 {password
												? 'text-[12px] !top-0 !left-2'
												: ''}"
										>
											{$i18n.t('Password')}
										</label>
									</div>

									<p
										class="text-sm -mt-1.5 pl-1 text-left text-gray-700/80 hover:text-gray-900 cursor-pointer"
									>
										Забыли пароль?
									</p>
								</div>

								<div class="mt-7">
									<button
										class="bg-gray-800/90 text-white w-[80%] mx-auto hover:bg-gray-900/70 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition rounded-2xl font-medium text-sm py-2.5"
										type="submit"
									>
										{$i18n.t('Sign in')}
									</button>

									<div class="mt-4 text-sm text-center text-gray-600/80">
										{$i18n.t('У вас нет аккаунта?')}
										<button
											class="font-medium underline text-gray-800/90 hover:text-gray-900"
											type="button"
											on:click={() => {
												mode = 'signup';
											}}
										>
											{$i18n.t('Создать')}
										</button>
									</div>
								</div>
							</form>
						{:else if mode === 'signup'}
							<form
								id="signup-form"
								name="signup-form"
								method="post"
								autocomplete="on"
								on:submit={submitHandler}
								class="flex flex-col justify-center"
							>
								<div class="mb-1">
									<div class="text-2xl font-medium">
										{$i18n.t(`Создать аккаунт`, { WEBUI_NAME: $WEBUI_NAME })}
									</div>
								</div>

								<div class="flex flex-col gap-3 mt-4">
									<div class="relative">
										<input
											bind:value={name}
											type="text"
											id="signup-name"
											name="name"
											class="peer block w-full rounded-2xl border py-3 px-4 text-neutral-900 focus:border-blue-500 dark:text-white shadow-xs outline-none border-neutral-200 dark-border-neutral-800 sm:text-sm sm:leading-6 transition-all"
											autocomplete="name"
											required
										/>
										<label
											for="signup-name"
											class="absolute left-2 text-sm top-1/2 -translate-y-1/2 text-neutral-500 bg-white dark:bg-neutral-800 rounded-lg transition-all duration-200 peer-focus:text-[12px] peer-focus:text-neutral-800 peer-focus:top-0 peer-focus:left-2 px-2 {name
												? 'text-[12px] !top-0 !left-2'
												: ''}"
										>
											{$i18n.t('Name')}
										</label>
									</div>

									<div class="relative">
										<input
											bind:value={email}
											type="email"
											id="signup-email"
											name="email"
											class="peer block w-full rounded-2xl border py-3 px-4 text-neutral-900 focus:border-blue-500 dark:text-white shadow-xs outline-none border-neutral-200 dark-border-neutral-800 sm:text-sm sm:leading-6 transition-all"
											autocomplete="email"
											required
										/>
										<label
											for="signup-email"
											class="absolute left-2 text-sm top-1/2 -translate-y-1/2 text-neutral-500 bg-white dark:bg-neutral-800 rounded-lg transition-all duration-200 peer-focus:text-[12px] peer-focus:text-neutral-800 peer-focus:top-0 peer-focus:left-2 px-2 {email
												? 'text-[12px] !top-0 !left-2'
												: ''}"
										>
											{$i18n.t('Email')}
										</label>
									</div>

									<div class="relative">
										<input
											bind:value={password}
											type="password"
											id="signup-password"
											name="password"
											class="peer block w-full rounded-2xl border py-3 px-4 pr-12 text-neutral-900 focus:border-blue-500 dark:text-white shadow-xs outline-none border-neutral-200 dark-border-neutral-800 sm:text-sm sm:leading-6 transition-all"
											autocomplete="new-password"
											required
										/>
										<label
											for="signup-password"
											class="absolute left-2 text-sm top-1/2 -translate-y-1/2 text-neutral-500 bg-white dark:bg-neutral-800 rounded-lg transition-all duration-200 peer-focus:text-[12px] peer-focus:text-neutral-800 peer-focus:top-0 peer-focus:left-2 px-2 {password
												? 'text-[12px] !top-0 !left-2'
												: ''}"
										>
											{$i18n.t('Password')}
										</label>
									</div>

									<!-- Add this code right after the password input field in the signup form -->
									{#if password}
										<!-- Password strength indicator -->
										<div class="px-1">
											<!-- Password strength progress bar -->
											<div class="w-full h-[6px] bg-gray-100 rounded-full dark:bg-gray-700 -mt-0.5">
												<div
													class="h-[6px] rounded-full transition-all duration-300"
													class:bg-red-500={passwordStrength < 40}
													class:bg-amber-500={passwordStrength >= 40 && passwordStrength < 100}
													class:bg-green-500={passwordStrength >= 100}
													style="width: {passwordStrength}%"
												></div>
											</div>
											<div class="flex items-center justify-end gap-2 mb-1 mt-0.5">
												{#if passwordStrength >= 100}
													<span class="text-xs font-medium text-green-600"
														>{$i18n.t('Надежный пароль')}</span
													>
												{:else}
													<span class="text-xs font-medium text-amber-600"
														>{$i18n.t('Ненадежный пароль')}</span
													>
												{/if}
											</div>
											<!-- <div class="flex flex-col gap-1 text-xs font-medium mt-3">
												<div class="flex items-center gap-4">
													{#if passwordChecks.minLength}
														<CheckIcon class="size-4 text-green-500" />
														<span class="text-green-600/90 dark:text-gray-300"
															>{$i18n.t('Длина от 8 символов')}</span
														>
													{:else}
														<X size={14} class="text-gray-400" />
														<span class="text-gray-400">{$i18n.t('Длина от 8 символов')}</span>
													{/if}
												</div>

												<div class="flex items-center gap-4">
													{#if passwordChecks.hasUppercase}
														<CheckIcon class="size-4 text-green-500" />
														<span class="text-green-600/90 dark:text-gray-300"
															>{$i18n.t('Заглавная буква')}</span
														>
													{:else}
														<X size={14} class="text-gray-400" />
														<span class="text-gray-400">{$i18n.t('Заглавная буква')}</span>
													{/if}
												</div>

												<div class="flex items-center gap-4">
													{#if passwordChecks.hasLowercase}
														<CheckIcon class="size-4 text-green-500" />
														<span class="text-green-600/90 dark:text-gray-300"
															>{$i18n.t('Строчная буква')}</span
														>
													{:else}
														<X size={14} class="text-gray-400" />
														<span class="text-gray-400">{$i18n.t('Строчная буква')}</span>
													{/if}
												</div>

												<div class="flex items-center gap-4">
													{#if passwordChecks.hasNumber}
														<CheckIcon class="size-4 text-green-500" />
														<span class="text-green-600/90 dark:text-gray-300"
															>{$i18n.t('Хотя бы одна цифра')}</span
														>
													{:else}
														<X size={14} class="text-gray-400" />
														<span class="text-gray-400">{$i18n.t('Хотя бы одна цифра')}</span>
													{/if}
												</div>

												<div class="flex items-center gap-4">
													{#if passwordChecks.hasSpecial}
														<CheckIcon class="size-4 text-green-500" />
														<span class="text-green-600/90 dark:text-gray-300"
															>{$i18n.t('Спецсимвол (!, @, #, ...)')}</span
														>
													{:else}
														<X size={14} class="text-gray-400" />
														<span class="text-gray-400">{$i18n.t('Спецсимвол (!, @, #, ...)')}</span
														>
													{/if}
												</div>
											</div> -->
										</div>
									{/if}
								</div>

								<div class="mt-5">
									<button
										class="bg-gray-800/90 text-white w-[80%] mx-auto hover:bg-gray-900/70 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition rounded-2xl font-medium text-sm py-2.5 disabled:opacity-50 disabled:cursor-not-allowed"
										type="submit"
										disabled={passwordStrength < 100}
									>
										{$i18n.t('Создать')}
									</button>

									<div class="mt-4 text-sm text-center text-gray-600/80">
										{$i18n.t('Уже есть аккаунт?')}
										<button
											class="font-medium underline text-gray-800/90 hover:text-gray-900"
											type="button"
											on:click={() => {
												mode = 'signin';
											}}
										>
											{$i18n.t('Sign in')}
										</button>
									</div>
								</div>
							</form>
						{/if}

						{#if $config?.features.enable_ldap && $config?.features.enable_login_form}
							<div class="mt-2">
								<button
									class="flex justify-center items-center text-xs w-full text-center underline"
									type="button"
									on:click={() => {
										if (mode === 'ldap')
											mode = ($config?.onboarding ?? false) ? 'signup' : 'signin';
										else mode = 'ldap';
									}}
								>
									<span
										>{mode === 'ldap'
											? $i18n.t('Continue with Email')
											: $i18n.t('Continue with LDAP')}</span
									>
								</button>
							</div>
						{/if}

						<div class="inline-flex items-center justify-center w-full mt-4">
							<hr class="w-28 h-px my-4 border-0 dark:bg-gray-100/40 bg-gray-700/10" />
							<span class="px-4 text-sm font-normal text-gray-700 dark:text-white bg-transparent"
								>{$i18n.t('or')}</span
							>

							<hr class="w-28 h-px my-4 border-0 dark:bg-gray-100/40 bg-gray-700/10" />
						</div>

						{#if Object.keys($config?.oauth?.providers ?? {}).length > 0}
							<div class="inline-flex items-center justify-center w-full">
								<hr class="w-32 h-px my-4 border-0 dark:bg-gray-100/10 bg-gray-700/10" />
								{#if $config?.features.enable_login_form || $config?.features.enable_ldap}
									<span
										class="px-3 text-sm font-medium text-gray-900 dark:text-white bg-transparent"
										>{$i18n.t('or')}</span
									>
								{/if}

								<hr class="w-32 h-px my-4 border-0 dark:bg-gray-100/10 bg-gray-700/10" />
							</div>
							<div class="flex flex-col space-y-2">
								{#if $config?.oauth?.providers?.google}
									<button
										class="flex justify-center items-center bg-gray-700/5 hover:bg-gray-700/10 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition w-full rounded-full font-medium text-sm py-2.5"
										on:click={() => {
											window.location.href = `${WEBUI_BASE_URL}/oauth/google/login`;
										}}
									>
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 48 48" class="size-6 mr-3">
											<path
												fill="#EA4335"
												d="M24 9.5c3.54 0 6.71 1.22 9.21 3.6l6.85-6.85C35.9 2.38 30.47 0 24 0 14.62 0 6.51 5.38 2.56 13.22l7.98 6.19C12.43 13.72 17.74 9.5 24 9.5z"
											/><path
												fill="#4285F4"
												d="M46.98 24.55c0-1.57-.15-3.09-.38-4.55H24v9.02h12.94c-.58 2.96-2.26 5.48-4.78 7.18l7.73 6c4.51-4.18 7.09-10.36 7.09-17.65z"
											/><path
												fill="#FBBC05"
												d="M10.53 28.59c-.48-1.45-.76-2.99-.76-4.59s.27-3.14.76-4.59l-7.98-6.19C.92 16.46 0 20.12 0 24c0 3.88.92 7.54 2.56 10.78l7.97-6.19z"
											/><path
												fill="#34A853"
												d="M24 48c6.48 0 11.93-2.13 15.89-5.81l-7.73-6c-2.15 1.45-4.92 2.3-8.16 2.3-6.26 0-11.57-4.22-13.47-9.91l-7.98 6.19C6.51 42.62 14.62 48 24 48z"
											/><path fill="none" d="M0 0h48v48H0z" />
										</svg>
										<span>{$i18n.t('Continue with {{provider}}', { provider: 'Google' })}</span>
									</button>
								{/if}
								{#if $config?.oauth?.providers?.microsoft}
									<button
										class="flex justify-center items-center bg-gray-700/5 hover:bg-gray-700/10 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition w-full rounded-full font-medium text-sm py-2.5"
										on:click={() => {
											window.location.href = `${WEBUI_BASE_URL}/oauth/microsoft/login`;
										}}
									>
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 21 21" class="size-6 mr-3">
											<rect x="1" y="1" width="9" height="9" fill="#f25022" /><rect
												x="1"
												y="11"
												width="9"
												height="9"
												fill="#00a4ef"
											/><rect x="11" y="1" width="9" height="9" fill="#7fba00" /><rect
												x="11"
												y="11"
												width="9"
												height="9"
												fill="#ffb900"
											/>
										</svg>
										<span>{$i18n.t('Continue with {{provider}}', { provider: 'Microsoft' })}</span>
									</button>
								{/if}
								{#if $config?.oauth?.providers?.github}
									<button
										class="flex justify-center items-center bg-gray-700/5 hover:bg-gray-700/10 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition w-full rounded-full font-medium text-sm py-2.5"
										on:click={() => {
											window.location.href = `${WEBUI_BASE_URL}/oauth/github/login`;
										}}
									>
										<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" class="size-6 mr-3">
											<path
												fill="currentColor"
												d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.92 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57C20.565 21.795 24 17.31 24 12c0-6.63-5.37-12-12-12z"
											/>
										</svg>
										<span>{$i18n.t('Continue with {{provider}}', { provider: 'GitHub' })}</span>
									</button>
								{/if}
								{#if $config?.oauth?.providers?.oidc}
									<button
										class="flex justify-center items-center bg-gray-700/5 hover:bg-gray-700/10 dark:bg-gray-100/5 dark:hover:bg-gray-100/10 dark:text-gray-300 dark:hover:text-white transition w-full rounded-full font-medium text-sm py-2.5"
										on:click={() => {
											window.location.href = `${WEBUI_BASE_URL}/oauth/oidc/login`;
										}}
									>
										<svg
											xmlns="http://www.w3.org/2000/svg"
											fill="none"
											viewBox="0 0 24 24"
											stroke-width="1.5"
											stroke="currentColor"
											class="size-6 mr-3"
										>
											<path
												stroke-linecap="round"
												stroke-linejoin="round"
												d="M15.75 5.25a3 3 0 0 1 3 3m3 0a6 6 0 0 1-7.029 5.912c-.563-.097-1.159.026-1.563.43L10.5 17.25H8.25v2.25H6v2.25H2.25v-2.818c0-.597.237-1.17.659-1.591l6.499-6.499c.404-.404.527-1 .43-1.563A6 6 0 1 1 21.75 8.25Z"
											/>
										</svg>

										<span
											>{$i18n.t('Continue with {{provider}}', {
												provider: $config?.oauth?.providers?.oidc ?? 'SSO'
											})}</span
										>
									</button>
								{/if}
							</div>
						{/if}

						{#if $config?.features.enable_ldap && $config?.features.enable_login_form}
							<div class="mt-2">
								<button
									class="flex justify-center items-center text-xs w-full text-center underline"
									type="button"
									on:click={() => {
										if (mode === 'ldap')
											mode = ($config?.onboarding ?? false) ? 'signup' : 'signin';
										else mode = 'ldap';
									}}
								>
									<span
										>{mode === 'ldap'
											? $i18n.t('Continue with Email')
											: $i18n.t('Continue with LDAP')}</span
									>
								</button>
							</div>
						{/if}
					</div>
				{/if}
			</div>
		</div>
	{/if}
</div>

<style>
	input:-webkit-autofill,
	input:-webkit-autofill:hover,
	input:-webkit-autofill:focus {
		-webkit-box-shadow: 0 0 0px 1000px white inset !important;
		-webkit-text-fill-color: inherit !important;
		transition: background-color 5000s ease-in-out 0s;
		background-color: transparent !important;
	}

	/* Для темной темы */
	.dark input:-webkit-autofill,
	.dark input:-webkit-autofill:hover,
	.dark input:-webkit-autofill:focus {
		-webkit-box-shadow: 0 0 0px 1000px #262626 inset !important;
		/* Используйте здесь цвет вашей темной темы */
	}
</style>
