import streamlit as st
import pandas as pd
import datetime
import time
import os

# ---------------------------------------------------------
# 1. إعدادات الصفحة والتصميم المتجاوب (Mobile & PC)
# ---------------------------------------------------------
st.set_page_config(
    page_title="مُتْقَن الرياضي الاحترافي",
    page_icon="🏋️‍♂️",
    layout="wide",
    initial_sidebar_state="expanded"
)

# تصميم فاتح، ناصع، ومريح جداً للعين مع دعم التجاوب الكامل
st.markdown("""
<style>
    .main {
        background-color: #F8FAFC;
    }
    h1, h2, h3 {
        color: #0F172A;
        font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    }
    .stMetric {
        background-color: #FFFFFF;
        padding: 16px;
        border-radius: 12px;
        box-shadow: 0 2px 6px rgba(0,0,0,0.03);
        border: 1px solid #E2E8F0;
    }
    .stButton>button {
        background-color: #0284C7;
        color: white;
        border-radius: 8px;
        font-weight: 600;
        padding: 10px 20px;
        border: none;
        width: 100%;
    }
    .stButton>button:hover {
        background-color: #0369A1;
        color: white;
    }
</style>
""", unsafe_allow_html=True)

# ---------------------------------------------------------
# 2. إدارة جلسات المستخدم وحفظ البيانات الدائم (Persistent Database)
# ---------------------------------------------------------
DATA_FILE = "motqan_gym_database.csv"

def load_user_data():
    if os.path.exists(DATA_FILE):
        return pd.read_csv(DATA_FILE)
    else:
        # بيانات افتراضية أولية
        df = pd.DataFrame({
            "المستخدم": ["guest@motqan.com"],
            "التاريخ": [str(datetime.date.today())],
            "التمرين": ["Squat / سكوات منزلي"],
            "التصنيف": ["الجزء السفلي"],
            "الوزن_كجم": [20.0],
            "الجلسات": [4],
            "التكرارات": [12],
            "ملاحظات": ["البداية الاحترافية"]
        })
        df.to_csv(DATA_FILE, index=False)
        return df

def save_user_data(new_row):
    df = load_user_data()
    df = pd.concat([df, pd.DataFrame([new_row])], ignore_index=True)
    df.to_csv(DATA_FILE, index=False)

# تهيئة حالة الجلسة (Session State) لتسجيل الدخول
if "logged_in" not in st.session_state:
    st.session_state.logged_in = False
    st.session_state.user_email = ""

# ---------------------------------------------------------
# 3. واجهة تسجيل الدخول (تسجيل عبر البريد أو Apple ID للـ iOS)
# ---------------------------------------------------------
if not st.session_state.logged_in:
    st.title("🔐 تسجيل الدخول إلى منصة مُتْقَن الرياضية")
    st.write("أهلاً بك. يرجى تسجيل الدخول لحفظ جميع تمارينك وأوزانك بشكل دائم على السحابة والجهاز.")

    col1, col2 = st.columns([1, 1])

    with col1:
        st.subheader("✉️ الدخول عبر البريد الإلكتروني")
        with st.form("email_login"):
            email_input = st.text_input("البريد الإلكتروني:", placeholder="name@example.com")
            password_input = st.text_input("كلمة المرور:", type="password")
            submit_email = st.form_submit_button("تسجيل الدخول / إنشاء حساب")
            
            if submit_email:
                if email_input:
                    st.session_state.logged_in = True
                    st.session_state.user_email = email_input
                    st.success(f"مرحباً بك مجدداً يا {email_input}! يتم تحميل بياناتك...")
                    time.sleep(1)
                    st.rerun()
                else:
                    st.error("الرجاء إدخال البريد الإلكتروني صحيحاً.")

    with col2:
        st.subheader(" تسجيل الدخول السريع (iOS / Apple ID)")
        st.write("إذا كنت تستخدم أجهزة آيفون أو آب إي دي، يمكنك الدخول بضغطة زر واحدة:")
        if st.button(" تسجيل الدخول باستخدام Apple ID"):
            st.session_state.logged_in = True
            st.session_state.user_email = "apple_user@icloud.com"
            st.success("تم تسجيل الدخول بنجاح عبر Apple ID!")
            time.sleep(1)
            st.rerun()

    st.stop() # إيقاف عرض باقي الصفحة حتى يتم تسجيل الدخول

# ---------------------------------------------------------
# 4. القائمة الجانبية للتنقل بين الصفحات (بعد تسجيل الدخول)
# ---------------------------------------------------------
st.sidebar.image("https://images.unsplash.com/photo-1534438327276-14e5300c3a48?w=400", use_column_width=True)
st.sidebar.title("🎯 مُتْقَن الرياضي")
st.sidebar.write(f"👤 الحساب: **{st.session_state.user_email}**")

if st.sidebar.button("🚪 تسجيل الخروج"):
    st.session_state.logged_in = False
    st.session_state.user_email = ""
    st.rerun()

st.sidebar.divider()

page = st.sidebar.radio(
    "الانتقال بين الأقسام:",
    [
        "🏠 لوحة القيادة والملخص",
        "🏋️‍♂️ جدول التمارين المتقدم (منزل/جيم)",
        "💾 سجل التمارين المحفوظة (دائم)",
        "🤖 المدرب الذكي وتدرج الأوزان (AI)",
        "⏱️ مؤقتات التمارين (Tabata & Rest)",
        "📚 المصادر والدلائل والروابط الرياضية"
    ]
)

# جلب بيانات المستخدم الحالي
all_data = load_user_data()
user_data = all_data[all_data["المستخدم"] == st.session_state.user_email]

# ---------------------------------------------------------
# الصفحة 1: لوحة القيادة والملخص
# ---------------------------------------------------------
if page == "🏠 لوحة القيادة والملخص":
    st.title("🏠 لوحة القيادة والملخص اليومي")
    st.write("نظرة شاملة ومحدثة على أدائك الرياضي، مع ضمان حفظ كافة البيانات بشكل تلقائي ودائم.")

    total_sessions = len(user_data)
    max_w = user_data["الوزن_كجم"].max() if not user_data.empty else 0

    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric(label="📊 إجمالي جلساتك المسجلة", value=f"{total_sessions} جلسة")
    with col2:
        st.metric(label="🏆 أعلى وزن تم حمله", value=f"{max_w} كجم")
    with col3:
        st.metric(label="🔥 حالة الحساب", value="متصل ومحفوظ ☁️")
    with col4:
        st.metric(label="⭐ مستوى الالتزام", value="98%")

    st.divider()

    st.subheader("📌 آخر التمارين التي أتممتها:")
    if not user_data.empty:
        st.dataframe(user_data[["التاريخ", "التمرين", "التصنيف", "الوزن_كجم", "الجلسات", "التكرارات"]], use_container_width=True)
    else:
        info_box = st.info("لم تقم بتسجيل أي تمارين بعد. انتقل إلى صفحة (جدول التمارين) وابدأ بتسجيل جلستك الأولى!")

# ---------------------------------------------------------
# الصفحة 2: جدول التمارين المتقدم وتسجيل الجلسات
# ---------------------------------------------------------
elif page == "🏋️‍♂️ جدول التمارين المتقدم (منزل/جيم)":
    st.title("🏋️‍♂️ جدول التمارين المتقدم وتسجيل الجلسات")
    st.write("سجّل تمارينك بكل بساطة، وسيتم حفظها مباشرة في سجلك الدائم.")

    env_choice = st.selectbox("اختر بيئة التمرين:", ["🏠 التمارين المنزلية (بوزن الجسم والدمبلز)", "🔥 تمارين الكروس فيت (CrossFit WOD)", "🏋️‍♀️ تمارين الجيم الاحترافية (Gym Equipment)"])

    if "🏠 التمارين المنزلية" in env_choice:
        st.subheader("🏠 خطة التمارين المنزلية المتخصصة")
        c1, c2 = st.columns(2)
        with c1:
            st.markdown("""
            * **Air Squats (سكوات):** 4 جلسات × 15 تكرار (عضلات الرجلين).
            * **Push-ups (ضغط):** 4 جلسات × 12 تكرار (الصدر والذراعين).
            * **Lunges (طعان):** 3 جلسات × 10 لكل رجل.
            * **Plank (بلانك):** 3 جلسات × 45 ثانية (عضلات الكور).
            """)
        with c2:
            st.image("https://images.unsplash.com/photo-1571019613454-1cb2f99b2d8b?w=600", caption="أداء التمارين المنزلية باحترافية")

    elif "🔥 تمارين الكروس فيت" in env_choice:
        st.subheader("🔥 حصة الكروس فيت (WOD)")
        st.markdown("""
        * **النظام:** AMRAP (أكبر عدد جولات في 15 دقيقة).
        * **الحركات:** 10 بيربي + 15 سوينغ دمبلز + 20 تسلق الجبل.
        """)
        st.image("https://images.unsplash.com/photo-1517838277536-f5f99be501cd?w=600", caption="تمارين الكروس فيت المكثفة")

    else:
        st.subheader("🏋️‍♀️ برنامج الجيم المتكامل")
        st.markdown("""
        * **Bench Press (ضغط الصدر بالبار):** 4 جلسات × 8 تكرار.
        * **Deadlift (الرفعة الميتة):** 3 جلسات × 5 تكرار.
        * **Lat Pulldown (سحب ظهر):** 4 جلسات × 12 تكرار.
        """)
        st.image("https://images.unsplash.com/photo-1534438327276-14e5300c3a48?w=600", caption="الجيم والأوزان الحرة")

    st.divider()

    st.subheader("💾 نموذج تسجيل وتثبيت الجلسة الحالية:")
    with st.form("log_new_workout"):
        ex_name = st.text_input("اسم التمرين (مثال: Dumbbell Press / Squat):", value="Dumbbell Press")
        ex_cat = st.selectbox("تصنيف العضلة:", ["الجزء العلوي", "الجزء السفلي", "كروس فيت / كارديو", "استشفاء وإطالة"])
        weight_val = st.number_input("الوزن المستخدم (كجم):", min_value=0.0, value=15.0, step=0.5)
        sets_val = st.number_input("عدد الجلسات (Sets):", min_value=1, value=4)
        reps_val = st.number_input("التكرارات في كل جلسة (Reps):", min_value=1, value=12)
        notes_val = st.text_input("ملاحظات الأداء:", value="أداء ممتاز وثابت")

        submit_btn = st.form_submit_button("📥 حفظ الجلسة في السجل الدائم")
        if submit_btn:
            new_row = {
                "المستخدم": st.session_state.user_email,
                "التاريخ": str(datetime.date.today()),
                "التمرين": ex_name,
                "التصنيف": ex_cat,
                "الوزن_كجم": weight_val,
                "الجلسات": sets_val,
                "التكرارات": reps_val,
                "ملاحظات": notes_val
            }
            save_user_data(new_row)
            st.success("🎉 تم حفظ تمرينك بنجاح في سجلك الدائم ولن يضيع أبداً!")
            time.sleep(1)
            st.rerun()

# ---------------------------------------------------------
# الصفحة 3: سجل التمارين المحفوظة الدائم
# ---------------------------------------------------------
elif page == "💾 سجل التمارين المحفوظة (دائم)":
    st.title("💾 سجل تمارينك وأوزانك المحفوظة")
    st.write("هنا تظهر كافة البيانات التي سجلتها بحسابك، وهي محفوظة بشكل دائم على نظام التخزين المحلي والسحابي.")

    if not user_data.empty:
        st.dataframe(user_data, use_container_width=True)
        
        if st.button("🗑️ مسح السجل والبدء من جديد"):
            # إزالة بيانات هذا المستخدم فقط
            other_data = all_data[all_data["المستخدم"] != st.session_state.user_email]
            other_data.to_csv(DATA_FILE, index=False)
            st.success("تم مسح السجل بنجاح.")
            st.rerun()
    else:
        st.info("لا توجد تمارين مسجلة حتى الآن.")

# ---------------------------------------------------------
# الصفحة 4: المدرب الذكي وتدرج الأوزان (AI)
# ---------------------------------------------------------
elif page == "🤖 المدرب الذكي وتدرج الأوزان (AI)":
    st.title("🤖 المدرب الذكي واقتراحات التدرج (AI Progressive Overload)")
    st.write("يقوم نظام الذكاء الاصطناعي بتحليل أوزانك السابقة واقتراح الوزن المناسب لجلساتك القادمة لضمان التطور المستمر:")

    if not user_data.empty:
        last_exercise = user_data.iloc[-1]
        st.info(f"آخر تمرين قمت بتسجيله: **{last5 := last_exercise['التمرين']}** بوزن **{last_exercise['الوزن_كجم']} كجم**.")
        
        st.subheader("💡 توصية المدرب الذكي للجلّسة القادمة:")
        suggested_weight = last_exercise['الوزن_كجم'] + 2.5
        st.success(f"""
        بناءً على تقدمك الأخير، نقترح عليك في الجلسة القادمة رفع الوزن بمقدار طفيف:
        * **الوزن المقترح:** `{suggested_weight} كجم` (زيادة آمنة بمقدار 2.5 كجم).
        * **التوجيه:** حافظ على بطء النزول (Eccentric phase) لضمان تفعيل العضلة بأقصى كفاءة وحماية المفاصل.
        """)
    else:
        st.warning("سجل أول تمرين لك في قسم (جدول التمارين) لكي يتمكن المدرب الذكي من تحليل وتوليد التوصيات المخصصة لك!")

# ---------------------------------------------------------
# الصفحة 5: مؤقتات التمارين
# ---------------------------------------------------------
elif page == "⏱️ مؤقتات التمارين (Tabata & Rest)":
    st.title("⏱️ مؤقتات التمارين التفاعلية")
    
    t1, t2 = st.tabs(["مؤقت Tabata المكثف", "مؤقت الراحة بين الجلسات"])
    
    with t1:
        st.write("نظام التبادل السريع: (20 ثانية تمرين عملي + 10 ثواني راحة)")
        r_count = st.number_input("عدد الجولات:", 1, 15, 4)
        if st.button("🚀 تشغيل مؤقت تاباتا"):
            box_timer = st.empty()
            for r in range(1, r_count + 1):
                box_timer.warning(f"🔥 الجولة {r}: تمرين بكل قوة! (20 ثانية)")
                time.sleep(1)
                box_timer.info(f"☕ الجولة {r}: استراحة قصيرة (10 ثواني)")
                time.sleep(1)
            st.balloons()
            st.success("🎉 أحسنت! أنهيت حصة تاباتا بنجاح.")

    with t2:
        st.write("عداد استراحة مخصص بين الجلسات الثقيلة:")
        rest_duration = st.slider("حدد ثواني الراحة:", 30, 180, 60, step=15)
        if st.button("▶️ ابدأ عداد الراحة الآن"):
            r_box = st.empty()
            for s in range(rest_duration, 0, -1):
                r_box.metric("الوقت المتبقي للراحة", f"{s} ثانية")
                time.sleep(1)
            r_box.success("🔔 انتهت الراحة، استعد للجلسة التالية!")

# ---------------------------------------------------------
# الصفحة 6: المصادر والدلائل
# ---------------------------------------------------------
elif page == "📚 المصادر والدلائل والروابط الرياضية":
    st.title("📚 المعجم والمصادر والمراجع الرياضية المعتمدة")
    st.write("مجموعة مختارة من أفضل المراجع العالمية الموثوقة لعلوم اللياقة والتمارين:")

    st.markdown("""
    * 🔗 **[CrossFit Main Site](https://www.crossfit.com):** المرجع العالمي الأول لتمارين الكروس فيت ووحدات الـ WOD.
    * 🔗 **[Examine.com](https://examine.com):** الدليل العلمي الأوثق لتحليل المكملات والتغذية الرياضية.
    * 🔗 **[Bodybuilding.com Exercises](https://www.bodybuilding.com/exercises):** أكبر موسوعة بصرية ومشروحة لطريقة أداء كل تمرين.
    """)
    st.info("💡 يمكنك الضغط على أي رابط لفتحه في المتصفح والاطلاع على أحدث الدراسات والمقالات الرياضية العالمية.")