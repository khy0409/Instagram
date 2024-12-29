<!-- eslint-disable vue/multi-word-component-names -->
<template>
  <div class="register_container">
    <h1 class="instagram_title">instagram</h1>
    <p class="register_prompt">친구들의 사진과 동영상을 보려면 가 <br />입하세요.</p>

    <div class="input_group">
      <input
        v-model="emailOrPhone"
        type="text"
        class="input_field"
        placeholder="휴대폰 번호 또는 이메일 주소"
        @blur="checkEmailOrPhone"
        @keyup.enter="register"
        @input="checkEmailOrPhoneDebounced"
      />
      <p v-if="emailOrPhoneError" class="error_message_2">{{ emailOrPhoneError }}</p>
    </div>

    <div class="input_group">
      <input
        v-model="password"
        type="password"
        class="input_field"
        placeholder="비밀번호"
        @blur="checkPasswordLength"
        @keyup.enter="register"
        @input="checkPasswordLengthDebounced"
      />
      <p v-if="passwordLengthError" class="error_message_2">{{ passwordLengthError }}</p>
    </div>

    <div class="input_group">
      <input
        v-model="password_check"
        type="password"
        class="input_field"
        placeholder="비밀번호 확인"
        @blur="checkPasswordMatch"
        @keyup.enter="register"
        @input="checkPasswordMatchDebounced"
      />
      <p v-if="passwordError" class="error_message_2">{{ passwordError }}</p>
    </div>

    <div class="input_group">
      <input
        v-model="name"
        type="text"
        class="input_field"
        placeholder="성명"
        @blur="checkName"
        @keyup.enter="register"
        @input="checkNameDebounced"
      />
      <p v-if="nameError" class="error_message_2">{{ nameError }}</p>
    </div>

    <div class="input_group">
      <input
        v-model="userName"
        type="text"
        class="input_field"
        placeholder="사용자 이름"
        @blur="validateUserName"
        @input="validateUserNameDebounced"
        @keyup.enter="register"
      />
      <p v-if="userNameError" class="error_message_2">{{ userNameError }}</p>
    </div>

    <div class="button">
      <button class="register_button" @click="register">가입</button>
    </div>

    <div class="login_prompt">
      계정이 있으신가요? <RouterLink to="/" class="login">로그인</RouterLink>
    </div>

    <!-- 모달 컴포넌트 추가 -->
    <modal-component
      :title="modalTitle"
      :message="modalMessage"
      :isOpen="isModalOpen"
      @closeModal="modalStatus(false)"
    />
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { RouterLink, useRouter } from 'vue-router'
import ModalComponent from '@/components/ModalConponent/Registermodalconponent.vue' // 모달 컴포넌트 가져오기

// firebase import
import { auth } from '../../firebase.ts' // Firebase Object Load
import { createUserWithEmailAndPassword, sendEmailVerification } from 'firebase/auth'
import { db } from '../../firebase.ts' // Firestore
import { doc, getDocs, setDoc, collection, query, where } from 'firebase/firestore'

// 입력 값들을 ref로 선언하여 반응형으로 관리
const emailOrPhone = ref('')
const name = ref('')
const userName = ref('')
const password = ref('')
const password_check = ref('')
const emailOrPhoneError = ref('')
const userNameError = ref('')
const passwordError = ref('')
const passwordLengthError = ref('')
const nameError = ref('')
const emailOrPhoneChecked = ref(true)
const userNameChecked = ref(true)
const router = useRouter()

// 모달 상태 관리
const isModalOpen = ref(false) // 모달 열림 상태
const modalTitle = ref('알림') // 모달 제목
const modalMessage = ref('') // 모달 메시지

// 모달 닫기 함수
const modalStatus = (isOpen) => {
  isModalOpen.value = isOpen
}

// 비밀번호 길이 체크 함수
const checkPasswordLength = () => {
  if (password.value && password.value.length < 6) {
    passwordLengthError.value = '6자 이상의 비밀번호를 만드세요.'
  } else {
    passwordLengthError.value = ''
  }
}

// 비밀번호 확인 입력 필드 에러 체크 함수
const checkPasswordMatch = () => {
  if (password_check.value && password.value !== password_check.value) {
    passwordError.value = '비밀번호가 일치하지 않습니다.'
  } else {
    passwordError.value = ''
  }
}

// 성명의 유효성을 체크하는 함수
const checkName = () => {
  const regExp = /[0-9!@#$%^&*()_+\-=[\]{};':"\\|,.<>/?]+/

  if (name.value && regExp.test(name.value)) {
    nameError.value = '성명에는 숫자나 특수 문자를 사용할 수 없습니다.'
  } else {
    nameError.value = '' // 유효할 경우 에러 초기화
  }
}

// 입력 값이 비어있는 경우, 에러 메시지 초기화 함수
const resetError = (errorRef, checkedRef, checkedValue) => {
  errorRef.value = ''
  checkedRef.value = checkedValue
}

// 에러 메시지 설정 함수
const setError = (errorRef, errorStr, checkedRef) => {
  errorRef.value = errorStr
  checkedRef.value = false
}

// 이메일 또는 전화번호 중복 체크 함수
const checkEmailOrPhone = async () => {
  // 입력 값이 비어있으면 에러 메시지 초기화
  if (!emailOrPhone.value) {
    resetError(emailOrPhoneError, emailOrPhoneChecked, true)
    return
  }

  // 전화번호가 숫자만 포함하고 있는지 체크
  const isPhoneNumber = /^[0-9]+$/.test(emailOrPhone.value)

  if (isPhoneNumber) {
    // 전화번호 길이 체크
    if (emailOrPhone.value.length !== 11) {
      setError(
        emailOrPhoneError,
        '휴대폰 번호가 정확하지 않습니다. 국가 번호를 포함하여 전체 전화번호를 입력해주세요.',
        emailOrPhoneChecked
      )
      return
    }
  } else {
    // 이메일 형식 체크
    const isValidEmail = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.(com)$/.test(emailOrPhone.value)

    if (!isValidEmail) {
      setError(emailOrPhoneError, '올바른 이메일 주소를 입력하세요.', emailOrPhoneChecked)
      return
    }
  }

  // 사용중인 email/phone인지 검증
  try {
    const isEmail = emailOrPhone.value.includes('@')
    const queryField = isEmail ? 'email' : 'phone'
    const queryValue = isEmail ? emailOrPhone.value : emailOrPhone.value.replace(/-/g, '') // 전화번호에서 하이픈 제거

    // 쿼리 실행
    const querySnapshot = await getDocs(
      query(collection(db, 'instagram_user'), where(queryField, '==', queryValue))
    )

    // 중복 체크
    if (!querySnapshot.empty) {
      setError(
        emailOrPhoneError,
        isEmail ? '이미 사용중인 이메일입니다.' : '이미 사용중인 전화번호입니다.',
        emailOrPhoneChecked
      )
    } else {
      resetError(emailOrPhoneError, emailOrPhoneChecked, true)
    }
  } catch (error) {
    console.error('중복 체크 오류:', error)
    setError(emailOrPhoneError, '오류가 발생했습니다. 다시 시도해주세요.', emailOrPhoneChecked)
  }
}

// 사용자 이름 검증 함수
const validateUserName = async () => {
  // 사용자 이름이 한글이 포함되어 있는지 확인
  const isKorean = /[가-힣]/.test(userName.value)
  const isValidUserName = /^[a-zA-Z0-9._]+$/.test(userName.value) // 영어, 숫자, 밑줄 및 마침표만 허용
  const isOnlyDigits = /^\d+$/.test(userName.value) // 숫자만 포함되어 있는지 확인

  // 사용자 이름이 비어있으면 에러 초기화
  if (!userName.value) {
    resetError(userNameError, userNameChecked, false)
    return
  }

  // 사용자 이름이 한글이 포함되어 있는지 또는 유효하지 않은 형식인지 확인
  if (isKorean || !isValidUserName) {
    setError(
      userNameError,
      '사용자 이름에는 영어, 숫자, 밑줄 및 마침표만 사용할 수 있습니다.',
      userNameChecked
    )
    return
  } else if (isOnlyDigits) {
    setError(userNameError, '사용자 이름에 숫자만 포함할 수는 없습니다.', userNameChecked)
    return
  } else if (userName.value.length < 3) {
    setError(userNameError, '사용자 이름은 3자 이상이어야 합니다.', userNameChecked)
    return
  }

  try {
    const querySnapshot = await getDocs(
      query(collection(db, 'instagram_user'), where('user_name', '==', userName.value))
    )

    // 쿼리 결과가 있으면 중복된 사용자 이름이 존재하는 것
    if (!querySnapshot.empty) {
      setError(userNameError, '이미 사용중인 사용자 이름입니다.', userNameChecked)
    } else {
      resetError(userNameError, userNameChecked, true)
    }
  } catch (error) {
    console.error('사용자 이름 중복 체크 오류:', error)
    setError(userNameError, '중복 체크 오류가 발생했습니다. 다시 시도해주세요.', userNameChecked)
  }
}

// 디바운스 관련 코드
let debounceTimer

const debounceFunction = (fn, delay) => {
  return function (...args) {
    clearTimeout(debounceTimer)
    debounceTimer = setTimeout(() => {
      fn(...args)
    }, delay)
  }
}

const checkEmailOrPhoneDebounced = debounceFunction(checkEmailOrPhone, 300)
const validateUserNameDebounced = debounceFunction(validateUserName, 300)
const checkPasswordLengthDebounced = debounceFunction(checkPasswordLength, 300)
const checkPasswordMatchDebounced = debounceFunction(checkPasswordMatch, 300)
const checkNameDebounced = debounceFunction(checkName, 300)

// 회원가입 처리 함수
const register = async () => {
  // 중복 클릭 방지를 위한 버튼 비활성화
  const registerButton = document.querySelector('.register_button')
  registerButton.disabled = true

  // 중복 체크 재확인
  await checkEmailOrPhone()
  await validateUserName()

  // 중복 체크 및 입력 값 검증
  if (
    !emailOrPhone.value ||
    !password.value ||
    !password_check.value ||
    !name.value ||
    !userName.value ||
    emailOrPhoneError.value ||
    userNameError.value ||
    passwordError.value ||
    !emailOrPhoneChecked.value ||
    !userNameChecked.value
  ) {
    modalMessage.value = '입력되지 않은 항목이 있거나 중복된 값이 있습니다.'
    modalStatus(true) // 모달 열기
    registerButton.disabled = false // 버튼 활성화
    return
  }

  try {
    // Firebase에서 회원가입 처리
    const userCredential = await createUserWithEmailAndPassword(
      auth,
      emailOrPhone.value,
      password.value
    )
    const user = userCredential.user

    const userRef = doc(db, 'instagram_user', user.uid)

    await setDoc(userRef, {
      id: user.uid,
      phone: emailOrPhone.value.includes('@') ? '' : emailOrPhone.value,
      email: emailOrPhone.value.includes('@') ? emailOrPhone.value : '',
      name: name.value,
      user_name: userName.value,
      accession_date: new Date(),
      status: 'Y'
    })

    // 인증 이메일 발송
    await sendEmailVerification(user)
    modalMessage.value = '이메일 인증 링크가 발송되었습니다. 이메일을 확인하여 인증을 완료해주세요.'
    modalStatus(true)
    setTimeout(() => {
      router.push('/') // 2초 후 로그인 화면으로 이동
    }, 2000)
  } catch (error) {
    console.error('회원가입 중 오류 발생:', error)
    modalMessage.value = '회원가입 중 오류가 발생했습니다. 다시 시도해 주세요.'
    modalStatus(true) // 오류 메시지 모달 열기
  } finally {
    // 버튼 활성화
    registerButton.disabled = false
  }
}
</script>

<style src="./Register.css"></style>
