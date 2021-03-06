<template>
    <div>
        <h1 class="title title-register">{{ $t('register.text') }}</h1>
        <p>{{ $t('register.welcome') }}</p>
        <p>{{ $t('register.intro') }}</p>

        <Form ref="registerForm" id="register-form" class="form-horizontal" :rules="registerRules" :model="player">
            <h2>{{ $t('register.personalinfo') }}</h2>

            <Form-item :label="$t('form.login')" :label-width="150" prop="username" required>
                <Input name="username"
                       :placeholder="$t('form.login')"
                       v-model="player.username"
                       type="text" />
            </Form-item>
            <Form-item :label="$t('form.email')" :label-width="150" prop="email" required>
                <Input name="email"
                       :placeholder="$t('form.email')"
                       v-model="player.email"
                       type="text" />
            </Form-item>
            <div>
                <div class="col-sm-offset-2 col-sm-10">
                    <Button type="primary" @click.prevent="handleSubmit()" long>{{ $t('register.text') }}</Button>
                </div>
            </div>
        </Form>
    </div>
</template>

<script type="text/ecmascript-6">
    import api from '~/services/api';
    import {isEmpty, entries} from '~/lib/utils';

    export default {
        middleware: 'anonymous',
        head() {
            return {
                title: this.$t('register.title'),
            };
        },
        data() {
            const validateEmail = (rule, value, callback) => {
                if (value === '') {
                    callback(new Error('field.empty'));
                } else {
                    if (this.player.email_confirm !== '') {
                        this.$refs.registerForm.validateField('email_confirm');
                    }
                    callback();
                }
            };

            return {
                player: {
                    username: '',
                    email: '',
                },
                registerRules: {
                    username: [
                        {type: 'string', required: true},
                    ],
                    email: [
                        {type: 'email', required: true},
                        {validator: validateEmail},
                    ],
                },
            };
        },
        methods: {
            handleSubmit() {
                this.$refs.registerForm.validate((valid) => {
                    if (valid) {
                        api.lostPassword({player_registration: this.player}).then(() => {
                            this.$Notice.success({
                                title: this.$t('notice.success'),
                                desc: this.$t('account.lost.description'),
                            });
                            this.$router.push('/');
                        }).catch((e) => {
                            if (!isEmpty(e.response.data.error)) {
                                /* eslint-disable no-restricted-syntax */
                                for (const [key, messages] of entries(e.response.data.error)) {
                                    this.$Notice.error({
                                        title: this.$t(key),
                                        desc: messages.join('.\n'),
                                    });
                                }
                                /* eslint-enable no-restricted-syntax */
                            } else {
                                this.$Notice.error({
                                    title: this.$t('notice.error'),
                                    desc: this.$t('error.generic'),
                                });
                            }
                        });
                    } else {
                        this.$Notice.error({
                            title: this.$t('notice.error'),
                            desc: this.$t('form.invalid'),
                        });
                    }
                });
            },
        },
    };
</script>
