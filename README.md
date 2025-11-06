// iso_graffiti - React single-file site (default export) // Usage instructions (read first): // 1) This component expects an API endpoint at /api/instagram that returns JSON: //    { "posts": [ { "id": "...", "media_url": "https://...", "permalink": "https://www.instagram.com/p/..", "caption": "...", "timestamp": "2025-11-05T12:00:00Z" }, ... ] } //    You can implement /api/instagram as a serverless function (Vercel/Netlify) that fetches Instagram Basic Display or Instagram Graph API and returns the mapped JSON. //    - For personal accounts: Instagram Basic Display API (requires manual token refresh every ~60 days). //    - For business/creator accounts: Instagram Graph API via a Facebook App (long-lived tokens available). //    If you can't create the API, use the supplied FALLBACK_POSTS below for a static preview. // 2) Deploy: this file can be used inside a React app (create-react-app, Next.js, Vite). Tailwind CSS expected. //    If you don't use Tailwind, replace classes with your preferred CSS. // 3) Auto-update behavior: The component fetches /api/instagram on mount, then polls every 60 seconds. //    It also caches in sessionStorage for fast load and offline view. // 4) If you want a pure static HTML version instead, tell me and I will generate it.

import React, { useEffect, useState } from 'react';

// --- Fallback data (used when API fails) --- const FALLBACK_POSTS = [ { id: 'f1', media_url: 'https://via.placeholder.com/1200x1200.png?text=iso_graffiti+1', permalink: '#', caption: 'مثال: عمل جرافيتي - iso_graffiti', timestamp: '2025-11-01T10:00:00Z' }, { id: 'f2', media_url: 'https://via.placeholder.com/1200x1200.png?text=iso_graffiti+2', permalink: '#', caption: 'مثال 2', timestamp: '2025-10-28T16:00:00Z' } ];

export default function IsoGraffitiSite() { const [posts, setPosts] = useState(() => { try { const cached = sessionStorage.getItem('iso_posts'); return cached ? JSON.parse(cached) : []; } catch (e) { return []; } }); const [loading, setLoading] = useState(posts.length === 0); const [error, setError] = useState(null); const [modalIndex, setModalIndex] = useState(-1);

useEffect(() => { let mounted = true; async function fetchPosts() { setLoading(true); setError(null); try { const res = await fetch('/api/instagram', { cache: 'no-store' }); if (!res.ok) throw new Error(API returned ${res.status}); const json = await res.json(); const newPosts = (json.posts || []).map(p => ({ id: p.id, media_url: p.media_url, permalink: p.permalink, caption: p.caption || '', timestamp: p.timestamp || '' })); if (mounted) { setPosts(newPosts.length ? newPosts : FALLBACK_POSTS); try { sessionStorage.setItem('iso_posts', JSON.stringify(newPosts.length ? newPosts : FALLBACK_POSTS)); } catch(e){} } } catch (err) { console.error('Failed to fetch /api/instagram', err); if (mounted) { setError('تعذر جلب المنشورات من إنستغرام — عرض نسخة محفوظة أو نموذج ثابت.'); if (!posts || posts.length === 0) setPosts(FALLBACK_POSTS); } } finally { if (mounted) setLoading(false); } }

fetchPosts();
// Poll every 60 seconds for updates
const interval = setInterval(fetchPosts, 60000);
return () => { mounted = false; clearInterval(interval); };

// eslint-disable-next-line react-hooks/exhaustive-deps }, []);

function openModal(index) { setModalIndex(index); document.body.style.overflow = 'hidden'; } function closeModal() { setModalIndex(-1); document.body.style.overflow = ''; }

return ( <div className="min-h-screen bg-gray-50 text-gray-900"> <header className="max-w-5xl mx-auto py-8 px-4 flex flex-col md:flex-row items-center justify-between"> <div className="flex items-center gap-4"> <div className="w-16 h-16 bg-gradient-to-br from-purple-600 to-pink-500 rounded-2xl flex items-center justify-center text-white font-extrabold text-xl">ISO</div> <div> <h1 className="text-2xl md:text-3xl font-bold">iso_graffiti</h1> <p className="text-sm text-gray-600">فنان جرافيتي — معرض أعمالي من إنستغرام</p> <div className="mt-2 flex gap-3"> <a className="text-sm underline" href="https://instagram.com/iso_graffiti" target="_blank" rel="noreferrer">Instagram</a> <a className="text-sm underline" href="#contact">تواصل</a> </div> </div> </div> <div className="mt-6 md:mt-0 text-right"> <p className="text-sm text-gray-600">موقع يعرض أحدث منشورات إنستغرام تلقائيًا</p> </div> </header>

<main className="max-w-5xl mx-auto px-4 pb-16">
    <section className="mb-8">
      <div className="rounded-lg p-6 bg-white shadow">
        <h2 className="text-xl font-semibold mb-2">حول iso_graffiti</h2>
        <p className="text-gray-700">مرحبًا — هذا معرض لأعمالي في الجرافيتي. أنشر أعمالاً جاهزة للشارع، مشاريع فنية، وعروض ورش. للمزيد زر حسابي على إنستغرام أو تواصل معي عبر البريد أدناه.</p>
        <div id="contact" className="mt-4">
          <p className="text-sm text-gray-600">بريد تواصل: <a href="mailto:contact@iso_graffiti.example" className="underline">contact@iso_graffiti.example</a></p>
        </div>
      </div>
    </section>

    <section>
      <h3 className="text-lg font-semibold mb-4">المعرض — أحدث المنشورات</h3>

      {loading && <div className="p-6 bg-white rounded shadow text-center">جارٍ التحميل...</div>}
      {error && <div className="p-4 mb-4 rounded bg-yellow-50 border border-yellow-200 text-yellow-700">{error}</div>}

      <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-3">
        {posts.map((post, i) => (
          <button key={post.id} onClick={() => openModal(i)} className="group relative block w-full aspect-square overflow-hidden rounded-lg bg-gray-100">
            <img src={post.media_url} alt={post.caption || 'iso_graffiti'} className="w-full h-full object-cover transform group-hover:scale-105 transition" />
            <div className="absolute bottom-2 left-2 right-2 opacity-0 group-hover:opacity-100 transition text-xs text-white bg-black/40 rounded p-1">{truncate(post.caption, 80)}</div>
          </button>
        ))}
      </div>

      {posts.length === 0 && !loading && (
        <div className="mt-4 p-6 bg-white rounded shadow text-center">لا توجد منشورات لعرضها الآن.</div>
      )}
    </section>
  </main>

  <footer className="py-6 border-t bg-white">
    <div className="max-w-5xl mx-auto px-4 text-center text-sm text-gray-600">© {new Date().getFullYear()} iso_graffiti — تصميم موقع تلقائي من منشورات إنستغرام</div>
  </footer>

  {/* Modal */}
  {modalIndex >= 0 && posts[modalIndex] && (
    <div className="fixed inset-0 z-50 flex items-center justify-center p-4">
      <div onClick={closeModal} className="absolute inset-0 bg-black/60 backdrop-blur-sm" />
      <div className="relative max-w-3xl w-full bg-white rounded-lg overflow-hidden shadow-lg">
        <div className="flex justify-between items-start p-3">
          <a className="text-sm underline" href={posts[modalIndex].permalink} target="_blank" rel="noreferrer">عرض على إنستغرام</a>
          <button onClick={closeModal} className="text-gray-600 px-3 py-1">إغلاق ✕</button>
        </div>
        <div className="p-2">
          <img src={posts[modalIndex].media_url} alt={posts[modalIndex].caption} className="w-full max-h-[70vh] object-contain" />
          <div className="p-4">
            <p className="text-sm text-gray-700 whitespace-pre-line">{posts[modalIndex].caption}</p>
            <p className="mt-2 text-xs text-gray-500">{formatDate(posts[modalIndex].timestamp)}</p>
          </div>
        </div>
      </div>
    </div>
  )}
</div>

); }

// --- Helpers --- function truncate(text, n) { if (!text) return ''; return text.length > n ? text.slice(0, n - 1) + '…' : text; }

function formatDate(iso) fuck all
