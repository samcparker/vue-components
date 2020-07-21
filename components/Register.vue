<template>
  <div>
      <div class="elevation-10 pa-10">
        <h1 class="text-center">REGISTER</h1>

        <v-text-field label="Email" v-model="email"></v-text-field>
        <v-expand-transition>   
            <v-alert v-show="emailErrors.length > 0" color="red lighten-1" class="px-2 py-1">
                <div v-for="(error, index) in emailErrors" :key="index">
                    <p style="font-size: 14px" class="white--text mb-0"><v-icon small color="white">mdi-close-thick</v-icon> {{ emailRequirements[error] }}</p>
                </div>
            </v-alert>
        </v-expand-transition>

        <v-text-field label="Username" v-model="username"></v-text-field>
        <v-expand-transition>   
            <v-alert v-show="usernameErrors.length > 0" color="red lighten-1" class="px-2 py-1">
                <div v-for="(error, index) in usernameErrors" :key="index">
                    <p style="font-size: 14px" class="white--text mb-0"><v-icon small color="white">mdi-close-thick</v-icon> {{ usernameRequirements[error] }}</p>
                </div>
            </v-alert>
        </v-expand-transition>

        <v-text-field label="Password" type="password" v-model="password"></v-text-field>
        <v-text-field label="Confirm Password" type="password" v-model="password2"></v-text-field>
        <v-expand-transition>   
            <v-alert v-show="passwordErrors.length > 0" color="red lighten-1" class="px-2 py-1">
                <div v-for="(error, index) in passwordErrors" :key="index">
                    <p style="font-size: 14px" class="white--text mb-0"><v-icon small color="white">mdi-close-thick</v-icon> {{ passwordRequirements[error] }}</p>
                </div>
            </v-alert>
        </v-expand-transition>
        <v-btn width="100%" :tile="tile" class="mb-4 mt-2" color="primary" @click="submit">REGISTER</v-btn>
        <v-divider class="my-5"></v-divider>
        <v-row class="mx-0">
            <v-col class="py-0 px-0 mb-2" cols="12">
                <p class="my-0 subtext text-center">Or use an alternative provider:</p>
            </v-col>
            <v-col cols="6" class="px-0 pr-2">
                <v-btn width="100%" :tile="tile" color="secondary"><v-icon left>mdi-google</v-icon>Google</v-btn>
            </v-col>
            <v-col cols="6" class="px-0 pl-2">
                <v-btn width="100%" :tile="tile" color="secondary"><v-icon left>mdi-facebook</v-icon>Facebook</v-btn>
            </v-col>
        </v-row>
      </div>
      <p class="mt-10 subtext text-center">Already have an account? <a href="/login">Login</a></p>
  </div>
</template>

<script>
import validator from "validator";
import passwordValidator from "password-validator";

const passwordSchema = new passwordValidator();

passwordSchema
.is().min(8)
.is().max(100)
.has().uppercase()
.has().lowercase()
.has().digits();

const usernameSchema = new passwordValidator();
usernameSchema
.is().min(3)
.is().max(20)
.has().not().symbols()
.has().not().spaces();

export default {
    methods: {
        submit() {
            // ensure details are not changed
            const cemail = this.email;
            const cpassword = this.password;
            const cpassword2 = this.password2;
            const cusername = this.username;

            var vm = this;

            this.isValid(cemail, cpassword, cpassword2, cusername).then(function(valid) {
                if (valid) {
                    vm.createAccount(cemail, cpassword);
                }
                else {

                }
            });


        },
        isValid(email, password, password2, username) {
            var vm = this;
            return new Promise((resolve, reject) => {
                this.validateUsername(username)
                .then(() => 
                vm.validatePassword(password, password2)
                .then(() => 
                vm.validateEmail(email)
                .then(() => {
                    if (this.passwordErrors.length > 0 || this.usernameErrors.length > 0 || this.emailErrors.length > 0) {
                        this.error = true;
                        resolve(false);
                    }
                    else {
                        resolve(true);
                    }
                })));
            });
        },
        createAccount(email, password) {
            this.$fireAuth.createUserWithEmailAndPassword(email, password)
            .then(function() {
                console.log(`Account created with email ${email} and password ${password}.`);
            })
            .catch(function(error) {
                console.log("Error when creating account: ");
                console.log(error);
            });
        },
        firestoreIsTaken(field, value) {
            var vm = this;
            return new Promise((resolve, reject) => {
                vm.$fireStore.collection("users").where(field, "==", value).get()
                .then(function(querySnapshot) {
                    console.log(querySnapshot.empty);
                    if (querySnapshot.empty) {
                        resolve(false);      
                    }
                    else {
                        resolve(true);
                    }
                })
                .catch(function(error) {
                    reject(error);
                });
            });
        },
        validateEmail(email) {
            return new Promise((resolve, reject) => {
                this.emailErrors = [];
    
                var vm = this;
    
                if (!validator.isEmail(email)) {
                    this.emailErrors.push("invalid");
                }
                
                // do not check if taken if there are errors
                if (this.emailErrors.length > 0) {
                    resolve(false);
                }
    
                this.firestoreIsTaken("email", email).then(function(taken) {
                    if (taken) {
                        vm.emailErrors.push("taken");
                        console.log(vm.emailErrors);
                        resolve(false);
                    }
                    else {
                        resolve(true);
                    }
                });
            });

        },
        validatePassword(password, password2) {
            return new Promise((resolve, reject) => {
                this.passwordErrors = [];
    
                // Check if passwords match
                if (password != password2) {
                    this.passwordErrors.push("matching");
                }
    
                // Check if password is legal
                var passValidation = passwordSchema.validate(password, { list: true });
                if (passValidation.length > 0) {
                    passValidation.forEach(v => {
                        console.log(v);
                        this.passwordErrors.push(v);
                    });
                }

                if (this.passwordErrors.length > 0) {
                    resolve(false);
                }
                else {
                    resolve(true);
                }
            });

        },
        validateUsername(username) {
            return new Promise((resolve, reject) => {
                var vm = this;
                this.usernameErrors = [];
    
                // Check if password is legal
                var usernameValidation = usernameSchema.validate(username, { list: true });
                if (usernameValidation.length > 0) {
                    usernameValidation.forEach(v => {
                        this.usernameErrors.push(v);
                    });
                }
    
                // do not check if taken if there are errors
                if (this.usernameErrors.length > 0) {
                    resolve(false);
                }
                // TODO : Check if username is taken
                this.firestoreIsTaken("username", username).then(function(taken) {
                    if (taken) {
                        vm.usernameErrors.push("taken");
                        resolve(false);
                    }
                    else {
                        resolve(true);
                    }
                });
            });

        }
    },
    props: {
        tile: {
            type: Boolean,
            default: false
        }
    },
    data() {
        return {
            error: false,
            email: "",
            username: "",
            password: "",
            password2: "",
            passwordRequirements: {
                "min": "Password min of 8.",
                "max": "Password max of 100.",
                "uppercase": "Password must contain uppercase.",
                "lowercase": "Password must contain lowercase.",
                "digits": "Password must contain digits.",
                "matching": "Passwords do not match.",
            },
            emailRequirements: {
                "invalid": "Email must be formatted correctly, e.g. joebloggs@email.com",
                "taken": "Email address is already in use."
            },
            usernameRequirements: {
                "min": "Username min of 3",
                "max": "Username max of 20",
                "symbols": "Username cannot contain symbols.",
                "spaces": "Username cannot contain spaces.",
                "taken": "Username is already taken."
            },
            passwordErrors: [],
            usernameErrors: [],
            emailErrors: [],
        }
    }
}


</script>

<style scoped>
.subtext {
    font-size: 14px;
    color: #555555;
}
</style>