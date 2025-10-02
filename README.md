<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF--8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>CheapWeb Store</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800;900&display=swap" rel="stylesheet">
    <style>
      @keyframes fadeInUp {
        from { opacity: 0; transform: translateY(20px); }
        to { opacity: 1; transform: translateY(0); }
      }
      .animate-fade-in-up { animation: fadeInUp 0.6s ease-out forwards; }
      @keyframes rain-fall {
        0% { transform: translateY(-50px); opacity: 0; }
        50% { opacity: 1; }
        100% { transform: translateY(350px); opacity: 0; }
      }
      .raindrop {
        position: absolute;
        top: 0;
        width: 1px;
        background: linear-gradient(to bottom, rgba(255, 255, 255, 0), rgba(255, 255, 255, 0.3));
        animation-name: rain-fall;
        animation-timing-function: linear;
        animation-iteration-count: infinite;
      }
    </style>
    <script>
      tailwind.config = {
        theme: {
          extend: {
            fontFamily: {
              sans: ['Inter', 'sans-serif'],
            },
            colors: {
              'primary': '#0ea5e9',
              'primary-hover': '#0284c7',
              'secondary': '#64748b',
              'background': '#020617',
              'surface': '#0f172a',
              'surface-accent': '#1e293b',
              'text-primary': '#f8fafc',
              'text-secondary': '#94a3b8',
              'accent': '#f59e0b',
              'border-color': '#334155',
              'success': '#22c55e',
              'danger': '#ef4444',
              'warning': '#eab308',
            },
            animation: {
              'fade-in-up': 'fadeInUp 0.6s ease-out forwards',
            },
            keyframes: {
              fadeInUp: {
                '0%': { opacity: '0', transform: 'translateY(20px)' },
                '100%': { opacity: '1', transform: 'translateY(0)' },
              },
            },
          }
        }
      }
    </script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.6.1/firebase-auth-compat.js"></script>
</head>
  <body class="bg-background text-text-primary font-sans">
    <div id="root"></div>
    <script type="text/babel">
      const { useState, useEffect, useCallback, useMemo } = React;

      // --- TYPES & ENUMS ---
      const View = {
        Home: 'HOME',
        Checkout: 'CHECKOUT',
        Login: 'LOGIN',
        Register: 'REGISTER',
        AdminDashboard: 'ADMIN_DASHBOARD',
        PrivacyPolicy: 'PRIVACY_POLICY',
        TermsOfService: 'TERMS_OF_SERVICE',
        ContactSupport: 'CONTACT_SUPPORT',
        Library: 'LIBRARY',
      };

      // --- CONSTANTS ---
      const ADMIN_EMAIL = 'admin@loliman.com';
      const MAX_FREE_DEMOS = 3;
      const PRODUCT_CATEGORIES = [
          'ALL',
          'BEST LANDINGPAGES',
          'ULTIMATE LOGIN UI',
          'FULL STACK WEB BUILD',
          'UI TEMPLATES',
          'CUSTOM UI TEMPLATES'
      ];
      const MOCK_PRODUCTS = [
          { id: 1, name: 'Quantum SaaS Landing Page', description: 'A futuristic and responsive landing page for any SaaS product.', price: 4499, rating: 5, category: 'BEST LANDINGPAGES', imageUrl: 'https://picsum.photos/seed/quantum/600/400', demoUrl: '#', downloadUrl: '#', discount: 50 },
          { id: 2, name: 'CyberAuth Login System UI', description: 'A secure and sleek login/signup interface with a modern, tech-inspired design.', price: 2799, rating: 5, category: 'ULTIMATE LOGIN UI', imageUrl: 'https://picsum.photos/seed/cyberauth/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 3, name: 'E-Commerce Pro Platform', description: 'A complete full-stack solution for online stores, including backend and database setup.', price: 24999, rating: 5, category: 'FULL STACK WEB BUILD', imageUrl: 'https://picsum.photos/seed/ecommpro/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 4, name: 'CreativeFolio UI Kit', description: 'A comprehensive UI kit for creative professionals to build stunning portfolios.', price: 3850, rating: 4, category: 'UI TEMPLATES', imageUrl: 'https://picsum.photos/seed/creativefolio/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 5, name: 'Bespoke Corporate Website UI', description: 'A custom-tailored UI template for a professional corporate presence.', price: 9500, rating: 5, category: 'CUSTOM UI TEMPLATES', imageUrl: 'https://picsum.photos/seed/bespoke/600/400', demoUrl: '#', downloadUrl: '#', discount: 40 },
          { id: 6, name: 'AeroLaunch Product Landing', description: 'A clean and high-converting landing page designed for product launches.', price: 4199, rating: 4, category: 'BEST LANDINGPAGES', imageUrl: 'https://picsum.photos/seed/aerolaunch/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 7, name: 'Minimalist Login Forms', description: 'A set of elegant and simple login UIs, focusing on excellent user experience.', price: 2100, rating: 5, category: 'ULTIMATE LOGIN UI', imageUrl: 'https://picsum.photos/seed/minimalist/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 8, name: 'ProjectFlow Management App', description: 'A full-stack project management tool to streamline team collaboration.', price: 29999, rating: 5, category: 'FULL STACK WEB BUILD', imageUrl: 'https://picsum.photos/seed/projectflow/600/400', demoUrl: '#', downloadUrl: '#' },
          { id: 9, name: 'Dashboard Widgets UI Pack', description: 'A versatile pack of widgets for building data-rich admin dashboards.', price: 5499, rating: 4, category: 'UI TEMPLATES', imageUrl: 'https://picsum.photos/seed/widgets/600/400', demoUrl: '#', downloadUrl: '#' }
      ];
      const MOCK_AD = { id: 1, title: 'Go Beyond Templates!', description: 'Need a fully custom digital solution? We build bespoke websites and applications from the ground up. Let\'s create something unique together.', link: '#' };
      const INITIAL_SITE_SETTINGS = { 
          upiId: 'your-upi-id@okhdfcbank',
          defaultDiscount: 33,
      };

      // --- UTILS ---
      const generateCaptcha = () => {
          const chars = 'ABCDEFGHJKLMNPQRSTUVWXYZ23456789';
          let captcha = '';
          for (let i = 0; i < 6; i++) {
              captcha += chars.charAt(Math.floor(Math.random() * chars.length));
          }
          return captcha;
      };

      // --- HOOKS ---
      function useLocalStorage(key, initialValue) {
        const [storedValue, setStoredValue] = useState(() => {
          try {
            const item = window.localStorage.getItem(key);
            return item ? JSON.parse(item) : initialValue;
          } catch (error) {
            console.error(error);
            return initialValue;
          }
        });

        useEffect(() => {
          try {
            window.localStorage.setItem(key, JSON.stringify(storedValue));
          } catch (error) {
            console.error(error);
          }
        }, [key, storedValue]);

        return [storedValue, setStoredValue];
      }

      // --- COMMON COMPONENTS ---
      const Logo = () => (
          <div className="flex items-center space-x-3 cursor-pointer group">
              <div className="p-2 bg-surface-accent rounded-lg group-hover:bg-primary transition-colors duration-300">
                  <svg width="24" height="24" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg" className="text-primary group-hover:text-white transition-colors duration-300">
                      <path d="M12 2L4 6V18L12 22L20 18V6L12 2Z" fill="currentColor" fillOpacity="0.1"/>
                      <path d="M20 6L12 10L4 6" stroke="currentColor" strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round"/>
                      <path d="M12 22V10" stroke="currentColor" strokeWidth="1.5" strokeLinecap="round" strokeLinejoin="round"/>
                      <path d="M4 6V18L12 22L20 18V6L12 2L4 6Z" stroke="currentColor" strokeWidth="1.5" strokeLinejoin="round"/>
                  </svg>
              </div>
              <span className="text-xl font-bold text-text-primary tracking-wide hidden sm:block">
                  CheapWeb Store
              </span>
          </div>
      );

      const StarRating = ({ rating }) => (
          <div className="flex items-center">
              {[...Array(5)].map((_, i) => (
                  <svg key={i} className={`w-5 h-5 ${i < rating ? 'text-accent' : 'text-gray-600'}`} fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                      <path d="M9.049 2.927c.3-.921 1.603-.921 1.902 0l1.07 3.292a1 1 0 00.95.69h3.462c.969 0 1.371 1.24.588 1.81l-2.8 2.034a1 1 0 00-.364 1.118l1.07 3.292c.3.921-.755 1.688-1.54 1.118l-2.8-2.034a1 1 0 00-1.175 0l-2.8 2.034c-.784.57-1.838-.197-1.539-1.118l1.07-3.292a1 1 0 00-.364-1.118L2.98 8.72c-.783-.57-.38-1.81.588-1.81h3.461a1 1 0 00.951-.69l1.07-3.292z" />
                  </svg>
              ))}
          </div>
      );

      const Captcha = ({ captcha, onRefresh }) => (
          <div className="flex items-center justify-between p-2 rounded-md bg-background border border-border-color">
              <span className="text-lg font-bold tracking-widest text-text-secondary select-none italic line-through">
                  {captcha}
              </span>
              <button type="button" onClick={onRefresh} className="text-primary hover:text-primary-hover" aria-label="Refresh captcha">
                  <svg xmlns="http://www.w3.org/2000/svg" className="h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
                      <path fillRule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.899 2.186l-1.414 1.414A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a5.002 5.002 0 009.282 3.122l1.414 1.414A7.002 7.002 0 014.101 16.899V19a1 1 0 01-2 0v-5a1 1 0 011-1h5a1 1 0 010 2H5.001z" clipRule="evenodd" />
                  </svg>
              </button>
          </div>
      );

      // --- LAYOUT COMPONENTS ---
      const Header = ({ navigateTo, user, onLogout }) => {
          const isAdmin = user?.email === ADMIN_EMAIL;
          return (
              <header className="bg-surface/80 backdrop-blur-lg sticky top-0 z-50 border-b border-border-color/50">
                  <nav className="container mx-auto px-4 sm:px-6 lg:px-8 py-3 flex justify-between items-center">
                      <div onClick={() => navigateTo(View.Home)}>
                          <Logo />
                      </div>
                      <div className="flex items-center space-x-2 md:space-x-4">
                          <button onClick={() => navigateTo(View.Home)} className="text-text-secondary hover:text-text-primary transition-colors duration-200 hidden sm:block px-3 py-2 rounded-md font-medium">Home</button>
                          {user ? (
                              <>
                                  {isAdmin ? (
                                      <button onClick={() => navigateTo(View.AdminDashboard)} className="text-text-secondary hover:text-text-primary transition-colors duration-200 px-3 py-2 rounded-md font-medium">Dashboard</button>
                                  ) : (
                                      <button onClick={() => navigateTo(View.Library)} className="text-text-secondary hover:text-text-primary transition-colors duration-200 px-3 py-2 rounded-md font-medium">Library</button>
                                  )}
                                  <button onClick={onLogout} className="bg-danger/80 hover:bg-danger text-white font-bold py-2 px-3 md:px-4 text-sm md:text-base rounded-md transition-colors duration-300">Logout</button>
                              </>
                          ) : (
                              <button onClick={() => navigateTo(View.Login)} className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-3 md:px-4 text-sm md:text-base rounded-md transition-colors duration-300">Login</button>
                          )}
                      </div>
                  </nav>
              </header>
          );
      };

      const Footer = ({ navigateTo }) => {
          const currentYear = new Date().getFullYear();
          return (
              <footer className="bg-surface mt-16 border-t border-border-color">
                  <div className="container mx-auto px-6 py-8">
                      <div className="grid grid-cols-1 md:grid-cols-3 gap-8 text-center md:text-left">
                          <div>
                              <h3 className="text-xl font-bold text-primary mb-2">CheapWeb Store</h3>
                              <p className="text-text-secondary">Your premier source for high-quality, pre-built web assets.</p>
                          </div>
                          <div>
                              <h3 className="font-semibold text-text-primary mb-3">Quick Links</h3>
                              <ul className="space-y-2">
                                  <li><button onClick={() => navigateTo(View.Home)} className="cursor-pointer text-text-secondary hover:text-primary transition-colors">Home</button></li>
                                  <li><button onClick={() => navigateTo(View.PrivacyPolicy)} className="cursor-pointer text-text-secondary hover:text-primary transition-colors">Privacy Policy</button></li>
                                  <li><button onClick={() => navigateTo(View.TermsOfService)} className="cursor-pointer text-text-secondary hover:text-primary transition-colors">Terms of Service</button></li>
                              </ul>
                          </div>
                          <div>
                              <h3 className="font-semibold text-text-primary mb-3">Support</h3>
                              <ul className="space-y-2">
                                  <li><button onClick={() => navigateTo(View.ContactSupport)} className="cursor-pointer text-text-secondary hover:text-primary transition-colors">Contact Support</button></li>
                              </ul>
                          </div>
                      </div>
                      <div className="mt-8 pt-6 border-t border-border-color text-center text-text-secondary text-sm">
                          <p>&copy; {currentYear} CheapWeb Store. All Rights Reserved.</p>
                      </div>
                  </div>
              </footer>
          );
      };

      // --- UI COMPONENTS ---
      const AdModal = ({ onWatchAd, onClose }) => (
          <div className="fixed inset-0 bg-black bg-opacity-75 flex items-center justify-center z-50 animate-fade-in-up p-4">
              <div className="bg-surface rounded-lg shadow-xl p-6 sm:p-8 max-w-sm w-full text-center border border-border-color">
                  <h3 className="text-xl sm:text-2xl font-bold mb-4 text-primary">Unlock Demo</h3>
                  <p className="text-text-secondary mb-6">You've used all your free demo views for this item. Watch a short ad to view it again!</p>
                  <div className="flex flex-col sm:flex-row justify-center gap-4">
                      <button onClick={onClose} className="bg-secondary hover:bg-gray-500 text-white font-bold py-2 px-6 rounded-md transition-colors w-full sm:w-auto">Maybe Later</button>
                      <button onClick={onWatchAd} className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-6 rounded-md transition-colors w-full sm:w-auto">Watch Ad</button>
                  </div>
              </div>
          </div>
      );

      const RainAnimation = () => {
          const drops = useMemo(() => {
              return Array.from({ length: 100 }).map((_, i) => {
                  const style = {
                      left: `${Math.random() * 100}%`,
                      animationDuration: `${0.5 + Math.random() * 0.5}s`,
                      animationDelay: `${Math.random() * 5}s`,
                      height: `${20 + Math.random() * 30}px`,
                  };
                  return <div key={i} className="raindrop" style={style}></div>;
              });
          }, []);

          return <div className="absolute inset-0 z-0 overflow-hidden rounded-lg">{drops}</div>;
      };

      // --- PRODUCT COMPONENTS ---
      const ProductCard = ({ product, onBuyNow, onViewDemo, demoViews, isPurchased, isPending, defaultDiscount }) => {
          const [showAdModal, setShowAdModal] = useState(false);
          const viewCount = demoViews[product.id] || 0;

          const handleViewDemoClick = () => {
              if (viewCount < MAX_FREE_DEMOS || isPurchased) {
                  onViewDemo(product.id);
                  window.open(product.demoUrl, '_blank', 'noopener,noreferrer');
              } else {
                  setShowAdModal(true);
              }
          };

          const handleWatchAd = () => {
              console.log("Simulating rewarded ad...");
              setShowAdModal(false);
              setTimeout(() => {
                  onViewDemo(product.id, true); // true to reset count
                  window.open(product.demoUrl, '_blank', 'noopener,noreferrer');
              }, 1500);
          };

          const discountPercent = product.discount ?? defaultDiscount;
          const mrp = useMemo(() => {
              if (discountPercent <= 0) return product.price * 1.5;
              return product.price / (1 - (discountPercent / 100));
          }, [product.price, discountPercent]);

          return (
              <>
                  <div className="bg-surface rounded-lg overflow-hidden border border-border-color/50 transform hover:-translate-y-2 transition-all duration-300 flex flex-col group relative p-1 bg-gradient-to-br from-surface to-surface-accent hover:from-primary/10 hover:to-surface-accent">
                      <div className="rounded-md overflow-hidden">
                          <div className="overflow-hidden relative">
                              <img className="w-full h-56 object-cover group-hover:scale-110 transition-transform duration-500" src={product.imageUrl} alt={product.name} />
                              <div className="absolute top-0 left-0 w-full h-full bg-black/20 group-hover:bg-black/40 transition-colors duration-300"></div>
                              <div className="absolute top-4 right-4 bg-danger text-white text-xs font-bold px-2 py-1 rounded-md shadow-lg">{Math.round(discountPercent)}% OFF</div>
                          </div>
                          <div className="p-5 flex flex-col flex-grow bg-surface">
                              <h3 className="text-lg font-bold mb-2 text-text-primary group-hover:text-primary transition-colors duration-300">{product.name}</h3>
                              <p className="text-sm text-text-secondary flex-grow mb-4">{product.description}</p>
                              <div className="flex justify-between items-center mt-2 mb-4">
                                  <div className="flex items-baseline gap-2">
                                      <span className="text-2xl font-bold text-primary">₹{product.price.toLocaleString('en-IN')}</span>
                                      <span className="text-sm text-text-secondary line-through">₹{mrp.toLocaleString('en-IN', { maximumFractionDigits: 0 })}</span>
                                  </div>
                                  <StarRating rating={product.rating} />
                              </div>
                              <div className="mt-auto flex justify-between items-center pt-4 border-t border-border-color">
                                  <button onClick={handleViewDemoClick} className="text-primary text-sm hover:underline font-semibold disabled:text-gray-500 disabled:no-underline" disabled={!product.demoUrl}>
                                      View Demo {viewCount > 0 && !isPurchased ? `(${viewCount}/${MAX_FREE_DEMOS})` : ''}
                                  </button>
                                  {isPurchased ? (
                                      <span className="font-bold text-success py-2 px-4 text-sm flex items-center gap-2">
                                          <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fillRule="evenodd" d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z" clipRule="evenodd" /></svg>
                                          Purchased
                                      </span>
                                  ) : isPending ? (
                                      <span className="font-bold text-warning py-2 px-4 text-sm">Pending Approval</span>
                                  ) : (
                                      <button onClick={() => onBuyNow(product)} className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-4 rounded-md transition-all duration-300 text-sm transform group-hover:scale-105 shadow-md group-hover:shadow-lg group-hover:shadow-primary/30">Buy Now</button>
                                  )}
                              </div>
                          </div>
                      </div>
                  </div>
                  {showAdModal && <AdModal onClose={() => setShowAdModal(false)} onWatchAd={handleWatchAd} />}
              </>
          );
      };

      // --- PAGE COMPONENTS ---
      const AnimatedBanner = ({ ad }) => (
          <div className="bg-gradient-to-br from-surface to-surface-accent rounded-lg p-8 md:p-12 text-center shadow-2xl relative overflow-hidden h-80 flex flex-col justify-center items-center border border-border-color/50">
              <RainAnimation />
              <div className="absolute inset-0 bg-[radial-gradient(ellipse_80%_80%_at_50%_-20%,rgba(14,165,233,0.3),rgba(255,255,255,0))] opacity-50"></div>
              <div className="relative z-10 animate-fade-in-up">
                  <h1 className="text-3xl md:text-5xl font-extrabold mb-3 bg-clip-text text-transparent bg-gradient-to-r from-text-primary to-text-secondary">{ad.title}</h1>
                  <p className="text-md md:text-lg max-w-2xl mx-auto text-text-secondary">{ad.description}</p>
              </div>
          </div>
      );

      const HomePage = ({ products, ad, onBuyNow, onViewDemo, demoViews, purchasedItems, pendingItems, siteSettings }) => {
          const [activeCategory, setActiveCategory] = useState('ALL');

          const filteredProducts = useMemo(() => {
              if (activeCategory === 'ALL') {
                  return products;
              }
              return products.filter(p => p.category === activeCategory);
          }, [products, activeCategory]);

          return (
              <div className="space-y-16 animate-fade-in-up">
                  <AnimatedBanner ad={ad} />
                  <div>
                      <div className="flex flex-wrap justify-center gap-2 md:gap-4 mb-12">
                          {PRODUCT_CATEGORIES.map(category => (
                              <button
                                  key={category}
                                  onClick={() => setActiveCategory(category)}
                                  className={`px-4 py-2 text-sm md:text-base font-semibold rounded-full transition-all duration-300 border-2 ${
                                      activeCategory === category
                                          ? 'bg-primary border-primary text-white shadow-lg shadow-primary/20'
                                          : 'bg-surface-accent border-transparent hover:border-primary/50 text-text-secondary hover:text-text-primary'
                                  }`}
                              >
                                  {category}
                              </button>
                          ))}
                      </div>
                      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 md:gap-8">
                          {filteredProducts.map(product => (
                              <ProductCard
                                  key={product.id}
                                  product={product}
                                  onBuyNow={onBuyNow}
                                  onViewDemo={onViewDemo}
                                  demoViews={demoViews}
                                  isPurchased={purchasedItems.includes(product.id)}
                                  isPending={pendingItems.includes(product.id)}
                                  defaultDiscount={siteSettings.defaultDiscount}
                              />
                          ))}
                      </div>
                  </div>
              </div>
          );
      };
      
      const formatCurrency = (amount) => `₹${amount.toLocaleString('en-IN', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;

      const CheckoutPage = ({ product, user, siteSettings, onBack, onRequestPurchase }) => {
        const [paymentRequested, setPaymentRequested] = useState(false);
        const [copied, setCopied] = useState(false);

        const billDetails = useMemo(() => {
          const finalPrice = product.price;
          const basePrice = finalPrice / 0.84;
          const gst = basePrice * 0.18;
          const serviceCharge = basePrice * 0.02;
          const subtotal = basePrice + gst + serviceCharge;
          const calculatedDiscount = subtotal - finalPrice;
          
          const discountPercent = product.discount ?? siteSettings.defaultDiscount;
          const mrp = discountPercent > 0 && discountPercent < 100 ? finalPrice / (1 - (discountPercent / 100)) : finalPrice * 1.5;

          return { finalPrice, mrp, basePrice, gst, serviceCharge, discount: calculatedDiscount };
        }, [product.price, product.discount, siteSettings.defaultDiscount]);
        
        const QR_CODE_URL = `https://api.qrserver.com/v1/create-qr-code/?size=200x200&data=upi://pay?pa=${siteSettings.upiId}&pn=CheapWeb Store&am=${billDetails.finalPrice.toFixed(2)}&cu=INR&bgcolor=0f172a&color=f8fafc`;

        const handleConfirmPayment = () => {
          if (!user) {
            alert("User session expired. Please log in again.");
            onBack();
            return;
          }
          onRequestPurchase({
            userId: user.uid,
            userEmail: user.email || 'N/A',
            productId: product.id,
            productName: product.name,
            price: billDetails.finalPrice,
          });
          setPaymentRequested(true);
        };

        const handleCopy = () => {
          navigator.clipboard.writeText(siteSettings.upiId);
          setCopied(true);
          setTimeout(() => setCopied(false), 2000);
        };

        if (paymentRequested) {
          return (
            <div className="max-w-md mx-auto bg-surface p-8 rounded-lg shadow-lg text-center animate-fade-in-up border border-border-color">
              <svg xmlns="http://www.w3.org/2000/svg" className="h-16 w-16 text-primary mx-auto mb-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
              </svg>
              <h2 className="text-2xl font-bold mb-2">Payment Under Review</h2>
              <p className="text-text-secondary mb-6">Your payment is being verified and will reflect within 24 hours. You can check the status in your library shortly.</p>
              <button onClick={onBack} className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-6 rounded-md transition-colors">Back to Home</button>
            </div>
          );
        }

        return (
          <div className="max-w-4xl mx-auto animate-fade-in-up">
            <button onClick={onBack} className="text-primary hover:underline mb-6 flex items-center gap-2">
              <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fillRule="evenodd" d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z" clipRule="evenodd" /></svg>
              Back to Products
            </button>
            <div className="grid md:grid-cols-5 gap-8 bg-surface p-6 sm:p-8 rounded-lg shadow-lg border border-border-color">
              <div className="md:col-span-3 order-2 md:order-1">
                <h2 className="text-2xl font-bold mb-6 text-text-primary border-b border-border-color pb-4">Order Summary</h2>
                <div className="space-y-4 text-sm">
                  <p className="text-lg font-semibold mb-4 text-primary">{product.name}</p>
                  <div className="flex justify-between"><span className="text-text-secondary">Maximum Retail Price (MRP)</span><span>{formatCurrency(billDetails.mrp)}</span></div>
                  <div className="flex justify-between"><span className="text-text-secondary">Base Price</span><span>{formatCurrency(billDetails.basePrice)}</span></div>
                  <div className="flex justify-between"><span className="text-text-secondary">GST (18%)</span><span>+ {formatCurrency(billDetails.gst)}</span></div>
                  <div className="flex justify-between"><span className="text-text-secondary">Service Charges (2%)</span><span>+ {formatCurrency(billDetails.serviceCharge)}</span></div>
                  <div className="flex justify-between text-success"><span className="font-semibold">Discount</span><span className="font-semibold">- {formatCurrency(billDetails.discount)}</span></div>
                  <div className="border-t border-border-color my-4"></div>
                  <div className="flex justify-between items-center font-bold text-lg pt-2">
                    <span className="text-text-primary">Total Payable</span>
                    <span className="text-primary text-2xl">{formatCurrency(billDetails.finalPrice)}</span>
                  </div>
                </div>
              </div>
              <div className="md:col-span-2 text-center order-1 md:order-2">
                <h2 className="text-2xl font-bold mb-4 text-text-primary">Pay with UPI</h2>
                <div className="flex justify-center mb-4"><img src={QR_CODE_URL} alt="UPI QR Code" className="rounded-lg"/></div>
                <p className="text-text-secondary text-sm">Scan with any UPI app</p>
                <div className="my-4 text-text-secondary font-semibold">OR</div>
                <div className="bg-background p-3 rounded-lg flex items-center justify-between border border-border-color">
                  <span className="font-mono text-xs md:text-sm break-all">{siteSettings.upiId}</span>
                  <button onClick={handleCopy} className="bg-primary text-sm hover:bg-primary-hover text-white font-bold py-1 px-3 rounded transition-colors duration-300 ml-2 flex-shrink-0">{copied ? 'Copied!' : 'Copy'}</button>
                </div>
                <button onClick={handleConfirmPayment} className="mt-6 w-full bg-success hover:bg-green-600 text-white font-bold py-3 px-4 rounded transition-colors duration-300">I have paid</button>
              </div>
            </div>
          </div>
        );
      };

      const LoginPage = ({ navigateTo }) => {
          const [email, setEmail] = useState('');
          const [password, setPassword] = useState('');
          const [captcha, setCaptcha] = useState('');
          const [userCaptcha, setUserCaptcha] = useState('');
          const [error, setError] = useState('');
          const [loading, setLoading] = useState(false);

          useEffect(() => {
              setCaptcha(generateCaptcha());
          }, []);

          const handleSubmit = async (e) => {
              e.preventDefault();
              if (!email || !password) {
                  setError('Email and password are required.');
                  return;
              }
              if (userCaptcha.toUpperCase() !== captcha) {
                  setError('Invalid captcha. Please try again.');
                  setCaptcha(generateCaptcha());
                  setUserCaptcha('');
                  return;
              }
              setLoading(true);
              setError('');
              try {
                  await firebase.auth().signInWithEmailAndPassword(email, password);
                  navigateTo(View.Home);
              } catch (err) {
                  setError(err.message || 'Failed to sign in. Please check your credentials.');
                  setCaptcha(generateCaptcha());
                  setUserCaptcha('');
              } finally {
                  setLoading(false);
              }
          };

          const refreshCaptcha = () => setCaptcha(generateCaptcha());

          return (
              <div className="max-w-md mx-auto mt-10 animate-fade-in-up p-4 sm:p-0">
                  <div className="bg-surface shadow-2xl rounded-lg px-8 pt-6 pb-8 mb-4 border border-border-color">
                      <h2 className="text-center text-3xl font-bold mb-6 text-text-primary">Welcome Back</h2>
                      <form onSubmit={handleSubmit}>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="email">Email</label>
                              <input
                                  className="shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="email" type="email" placeholder="you@example.com" value={email} onChange={(e) => setEmail(e.target.value)} required
                              />
                          </div>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="password">Password</label>
                              <input
                                  className="shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="password" type="password" placeholder="******************" value={password} onChange={(e) => setPassword(e.target.value)} required
                              />
                          </div>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="captcha-input">Enter Captcha</label>
                              <Captcha captcha={captcha} onRefresh={refreshCaptcha} />
                              <input
                                  className="mt-2 shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="captcha-input" type="text" placeholder="Enter text from image" value={userCaptcha} onChange={(e) => setUserCaptcha(e.target.value)} required
                              />
                          </div>
                          {error && <p className="text-danger text-xs italic mb-4">{error}</p>}
                          <div className="flex items-center justify-between">
                              <button
                                  className="w-full bg-primary hover:bg-primary-hover text-white font-bold py-3 px-4 rounded focus:outline-none focus:shadow-outline transition-colors duration-300 disabled:bg-gray-500"
                                  type="submit" disabled={loading}
                              >
                                  {loading ? 'Signing In...' : 'Sign In'}
                              </button>
                          </div>
                          <p className="text-center text-text-secondary text-sm mt-6">
                              Don't have an account?{' '}
                              <button type="button" onClick={() => navigateTo(View.Register)} className="font-bold text-primary hover:underline">
                                  Sign Up
                              </button>
                          </p>
                      </form>
                  </div>
              </div>
          );
      };

      const RegisterPage = ({ navigateTo }) => {
          const [email, setEmail] = useState('');
          const [password, setPassword] = useState('');
          const [captcha, setCaptcha] = useState('');
          const [userCaptcha, setUserCaptcha] = useState('');
          const [error, setError] = useState('');
          const [loading, setLoading] = useState(false);

          useEffect(() => {
              setCaptcha(generateCaptcha());
          }, []);

          const handleSubmit = async (e) => {
              e.preventDefault();
              if (!email || !password) {
                  setError('Email and password are required.');
                  return;
              }
              if (password.length < 6) {
                  setError('Password should be at least 6 characters.');
                  return;
              }
              if (userCaptcha.toUpperCase() !== captcha) {
                  setError('Invalid captcha. Please try again.');
                  setCaptcha(generateCaptcha());
                  setUserCaptcha('');
                  return;
              }
              setLoading(true);
              setError('');
              try {
                  await firebase.auth().createUserWithEmailAndPassword(email, password);
                  navigateTo(View.Home);
              } catch (err) {
                  setError(err.message || 'Failed to register. Please try again.');
                  setCaptcha(generateCaptcha());
                  setUserCaptcha('');
              } finally {
                  setLoading(false);
              }
          };

          const refreshCaptcha = () => setCaptcha(generateCaptcha());

          return (
              <div className="max-w-md mx-auto mt-10 animate-fade-in-up p-4 sm:p-0">
                  <div className="bg-surface shadow-2xl rounded-lg px-8 pt-6 pb-8 mb-4 border border-border-color">
                      <h2 className="text-center text-3xl font-bold mb-6 text-text-primary">Create an Account</h2>
                      <form onSubmit={handleSubmit}>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="email-register">Email</label>
                              <input
                                  className="shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="email-register" type="email" placeholder="you@example.com" value={email} onChange={(e) => setEmail(e.target.value)} required
                              />
                          </div>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="password-register">Password</label>
                              <input
                                  className="shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="password-register" type="password" placeholder="Minimum 6 characters" value={password} onChange={(e) => setPassword(e.target.value)} required
                              />
                          </div>
                          <div className="mb-4">
                              <label className="block text-text-secondary text-sm font-bold mb-2" htmlFor="captcha-input-register">Enter Captcha</label>
                              <Captcha captcha={captcha} onRefresh={refreshCaptcha} />
                              <input
                                  className="mt-2 shadow appearance-none border rounded w-full py-3 px-4 bg-background border-border-color text-text-primary leading-tight focus:outline-none focus:ring-2 focus:ring-primary"
                                  id="captcha-input-register" type="text" placeholder="Enter text from image" value={userCaptcha} onChange={(e) => setUserCaptcha(e.target.value)} required
                              />
                          </div>
                          {error && <p className="text-danger text-xs italic mb-4">{error}</p>}
                          <div className="flex items-center justify-between">
                              <button
                                  className="w-full bg-primary hover:bg-primary-hover text-white font-bold py-3 px-4 rounded focus:outline-none focus:shadow-outline transition-colors duration-300 disabled:bg-gray-500"
                                  type="submit" disabled={loading}
                              >
                                  {loading ? 'Creating Account...' : 'Sign Up'}
                              </button>
                          </div>
                          <p className="text-center text-text-secondary text-sm mt-6">
                              Already have an account?{' '}
                              <button type="button" onClick={() => navigateTo(View.Login)} className="font-bold text-primary hover:underline">
                                  Login
                              </button>
                          </p>
                      </form>
                  </div>
              </div>
          );
      };

      const LibraryPage = ({ allProducts, purchasedIds, onBack }) => {
          const purchasedProducts = useMemo(() => {
              return allProducts.filter(p => purchasedIds.includes(p.id));
          }, [allProducts, purchasedIds]);

          return (
              <div className="max-w-6xl mx-auto animate-fade-in-up">
                  <button onClick={onBack} className="text-primary hover:underline mb-6 flex items-center gap-2">
                      <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fillRule="evenodd" d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z" clipRule="evenodd" /></svg>
                      Back to Home
                  </button>
                  <h1 className="text-3xl font-bold mb-8 border-b border-border-color pb-4">My Library</h1>
                  {purchasedProducts.length > 0 ? (
                      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
                          {purchasedProducts.map(product => (
                              <div key={product.id} className="bg-surface rounded-lg overflow-hidden shadow-lg border border-border-color flex flex-col">
                                  <img className="w-full h-56 object-cover" src={product.imageUrl} alt={product.name} />
                                  <div className="p-6 flex flex-col flex-grow">
                                      <h3 className="text-xl font-bold mb-2 text-text-primary">{product.name}</h3>
                                      <div className="flex-grow"></div>
                                      <a
                                          href={product.downloadUrl}
                                          download
                                          className="mt-4 w-full text-center bg-success hover:bg-green-600 text-white font-bold py-3 px-4 rounded-md transition-colors duration-300 flex items-center justify-center gap-2"
                                      >
                                          <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
                                              <path fillRule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414 0L9 10.586V3a1 1 0 112 0v7.586l1.293-1.293a1 1 0 111.414 1.414l-3 3a1 1 0 01-1.414 0l-3-3a1 1 0 010-1.414z" clipRule="evenodd" />
                                          </svg>
                                          Download Files
                                      </a>
                                  </div>
                              </div>
                          ))}
                      </div>
                  ) : (
                      <div className="text-center py-16 bg-surface rounded-lg border border-border-color">
                          <h2 className="text-2xl font-semibold mb-2">Your Library is Empty</h2>
                          <p className="text-text-secondary">Purchased items will appear here.</p>
                      </div>
                  )}
              </div>
          );
      };

      const PolicyPage = ({ title, children, onBack }) => (
          <div className="max-w-4xl mx-auto bg-surface p-6 sm:p-8 rounded-lg shadow-lg border border-border-color animate-fade-in-up">
              <button onClick={onBack} className="text-primary hover:underline mb-6 flex items-center gap-2">
                  <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor"><path fillRule="evenodd" d="M9.707 16.707a1 1 0 01-1.414 0l-6-6a1 1 0 010-1.414l6-6a1 1 0 011.414 1.414L5.414 9H17a1 1 0 110 2H5.414l4.293 4.293a1 1 0 010 1.414z" clipRule="evenodd" /></svg>
                  Back to Home
              </button>
              <h1 className="text-3xl font-bold mb-6 border-b border-border-color pb-4 text-text-primary">{title}</h1>
              <div className="prose prose-invert max-w-none text-text-secondary space-y-4">
                  {children}
              </div>
          </div>
      );

      // --- ADMIN COMPONENTS ---
      const AdManagement = ({ ad, setAd }) => {
          const [adData, setAdData] = useState(ad);
          const [saved, setSaved] = useState(false);

          useEffect(() => setAdData(ad), [ad]);

          const handleChange = (e) => {
              const { name, value } = e.target;
              setAdData(prev => ({ ...prev, [name]: value }));
          };

          const handleSubmit = (e) => {
              e.preventDefault();
              setAd(adData);
              setSaved(true);
              setTimeout(() => setSaved(false), 2000);
          };

          return (
              <div>
                  <h2 className="text-2xl font-semibold mb-4 text-text-primary">Advertisement</h2>
                  <form onSubmit={handleSubmit} className="space-y-4">
                      <input type="text" name="title" value={adData.title} onChange={handleChange} placeholder="Ad Title" className="w-full p-2 bg-background rounded border border-border-color focus:ring-primary focus:border-primary" required />
                      <textarea name="description" value={adData.description} onChange={handleChange} placeholder="Ad Description" className="w-full p-2 bg-background rounded border border-border-color focus:ring-primary focus:border-primary" required />
                      <input type="text" name="link" value={adData.link} onChange={handleChange} placeholder="Ad Link URL (not currently used)" className="w-full p-2 bg-background rounded border border-border-color focus:ring-primary focus:border-primary" required />
                      <div className="flex items-center space-x-4">
                          <button type="submit" className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-4 rounded">Save Ad</button>
                          {saved && <span className="text-success">Advertisement saved!</span>}
                      </div>
                  </form>
              </div>
          );
      };

      const OrderManagement = ({ pendingOrders, setPendingOrders, setAllUserProfiles }) => {
          const handleApprove = (orderId) => {
              const orderToApprove = pendingOrders.find(o => o.id === orderId);
              if (!orderToApprove) return;

              setAllUserProfiles(prevProfiles => {
                  const userProfile = prevProfiles[orderToApprove.userId] || { purchasedItems: [], demoViews: {} };
                  const updatedProfile = {
                      ...userProfile,
                      purchasedItems: [...new Set([...userProfile.purchasedItems, orderToApprove.productId])]
                  };
                  return { ...prevProfiles, [orderToApprove.userId]: updatedProfile };
              });

              setPendingOrders(prevOrders => prevOrders.map(order =>
                  order.id === orderId ? { ...order, status: 'approved' } : order
              ));
          };

          const handleReject = (orderId) => {
              setPendingOrders(prevOrders => prevOrders.map(order =>
                  order.id === orderId ? { ...order, status: 'rejected' } : order
              ));
          };

          const pending = pendingOrders.filter(o => o.status === 'pending');

          return (
              <div>
                  <h2 className="text-2xl font-semibold mb-4 text-text-primary">Pending Orders</h2>
                  {pending.length === 0 ? (
                      <p className="text-text-secondary">No pending orders at the moment.</p>
                  ) : (
                      <div className="overflow-x-auto">
                          <table className="min-w-full text-left">
                              <thead className="bg-surface-accent">
                                  <tr>
                                      <th className="py-2 px-4">Date</th>
                                      <th className="py-2 px-4 hidden sm:table-cell">User</th>
                                      <th className="py-2 px-4">Product</th>
                                      <th className="py-2 px-4 hidden md:table-cell">Amount</th>
                                      <th className="py-2 px-4">Actions</th>
                                  </tr>
                              </thead>
                              <tbody>
                                  {pending.map(order => (
                                      <tr key={order.id} className="border-b border-border-color hover:bg-surface-accent">
                                          <td className="py-2 px-4 text-sm">{new Date(order.date).toLocaleString()}</td>
                                          <td className="py-2 px-4 hidden sm:table-cell text-sm">{order.userEmail}</td>
                                          <td className="py-2 px-4">{order.productName}</td>
                                          <td className="py-2 px-4 hidden md:table-cell">₹{order.price.toLocaleString('en-IN')}</td>
                                          <td className="py-2 px-4 space-x-2">
                                              <button onClick={() => handleApprove(order.id)} className="bg-success hover:bg-green-600 text-white font-bold py-1 px-3 rounded text-sm">Approve</button>
                                              <button onClick={() => handleReject(order.id)} className="bg-danger hover:bg-red-600 text-white font-bold py-1 px-3 rounded text-sm">Reject</button>
                                          </td>
                                      </tr>
                                  ))}
                              </tbody>
                          </table>
                      </div>
                  )}
              </div>
          );
      };

      const ProductManagement = ({ products, setProducts }) => {
          const ProductForm = ({ product, onSave, onCancel }) => {
              const [formData, setFormData] = useState(
                product || { name: '', description: '', price: 0, rating: 3, category: PRODUCT_CATEGORIES[1], imageUrl: '', demoUrl: '', downloadUrl: '', discount: 0 }
              );

              const handleChange = (e) => {
                const { name, value } = e.target;
                setFormData(prev => ({ ...prev, [name]: (name === 'price' || name === 'rating' || name === 'discount') ? parseFloat(value) || 0 : value }));
              };

              const handleSubmit = (e) => {
                e.preventDefault();
                onSave({ ...formData, id: product?.id ?? Date.now() });
              };

              return (
                <form onSubmit={handleSubmit} className="bg-background p-4 rounded-md mt-4 space-y-4 border border-border-color">
                  <h3 className="text-xl font-bold">{product ? 'Edit Product' : 'Add New Product'}</h3>
                  <input type="text" name="name" value={formData.name} onChange={handleChange} placeholder="Product Name" className="w-full p-2 bg-surface rounded border border-border-color focus:ring-primary focus:border-primary" required />
                  <textarea name="description" value={formData.description} onChange={handleChange} placeholder="Description" className="w-full p-2 bg-surface rounded border border-border-color" required />
                  <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                    <input type="number" name="price" value={formData.price} onChange={handleChange} placeholder="Price (INR)" className="w-full p-2 bg-surface rounded border border-border-color" required />
                    <input type="number" name="rating" min="1" max="5" value={formData.rating} onChange={handleChange} placeholder="Rating (1-5)" className="w-full p-2 bg-surface rounded border border-border-color" required />
                    <select name="category" value={formData.category} onChange={handleChange} className="w-full p-2 bg-surface rounded border border-border-color">
                      {PRODUCT_CATEGORIES.slice(1).map(cat => <option key={cat} value={cat}>{cat}</option>)}
                    </select>
                    <input type="number" name="discount" min="0" max="99" value={formData.discount || ''} onChange={handleChange} placeholder="Discount % (optional)" className="w-full p-2 bg-surface rounded border border-border-color" />
                  </div>
                  <input type="text" name="imageUrl" value={formData.imageUrl} onChange={handleChange} placeholder="Image URL" className="w-full p-2 bg-surface rounded border border-border-color" required />
                  <input type="text" name="demoUrl" value={formData.demoUrl} onChange={handleChange} placeholder="Demo URL" className="w-full p-2 bg-surface rounded border border-border-color" required />
                  <input type="text" name="downloadUrl" value={formData.downloadUrl} onChange={handleChange} placeholder="Download File URL" className="w-full p-2 bg-surface rounded border border-border-color" required />
                  <div className="flex justify-end space-x-2">
                    <button type="button" onClick={onCancel} className="bg-secondary hover:bg-gray-600 text-white font-bold py-2 px-4 rounded">Cancel</button>
                    <button type="submit" className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-4 rounded">Save</button>
                  </div>
                </form>
              );
          }

          const [editingProduct, setEditingProduct] = useState(null);
          const [isFormVisible, setIsFormVisible] = useState(false);

          const handleAddNew = () => { setEditingProduct(null); setIsFormVisible(true); };
          const handleEdit = (product) => { setEditingProduct(product); setIsFormVisible(true); };
          const handleDelete = (productId) => { if (window.confirm('Are you sure you want to delete this product?')) { setProducts(products.filter(p => p.id !== productId)); } };
          const handleSaveProduct = (product) => {
            if (editingProduct) { setProducts(products.map(p => p.id === product.id ? product : p)); }
            else { setProducts([...products, product]); }
            setIsFormVisible(false); setEditingProduct(null);
          };

          return (
            <div>
              <div className="flex flex-col sm:flex-row justify-between items-start sm:items-center mb-4 gap-4">
                <h2 className="text-2xl font-semibold text-text-primary">Products</h2>
                <button onClick={handleAddNew} className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-4 rounded w-full sm:w-auto">Add New Product</button>
              </div>
              {isFormVisible && (<ProductForm product={editingProduct} onSave={handleSaveProduct} onCancel={() => { setIsFormVisible(false); setEditingProduct(null); }} />)}
              <div className="overflow-x-auto mt-6">
                <table className="min-w-full text-left">
                  <thead className="bg-surface-accent"><tr><th className="py-2 px-4">Name</th><th className="py-2 px-4 hidden sm:table-cell">Price</th><th className="py-2 px-4 hidden md:table-cell">Category</th><th className="py-2 px-4">Actions</th></tr></thead>
                  <tbody>
                    {products.map(product => (
                      <tr key={product.id} className="border-b border-border-color hover:bg-surface-accent">
                        <td className="py-2 px-4">{product.name}</td>
                        <td className="py-2 px-4 hidden sm:table-cell">₹{product.price.toLocaleString('en-IN')}</td>
                        <td className="py-2 px-4 hidden md:table-cell">{product.category}</td>
                        <td className="py-2 px-4 space-x-2"><button onClick={() => handleEdit(product)} className="text-primary hover:underline">Edit</button><button onClick={() => handleDelete(product.id)} className="text-danger hover:underline">Delete</button></td>
                      </tr>
                    ))}
                  </tbody>
                </table>
              </div>
            </div>
          );
      };

      const SettingsManagement = ({ siteSettings, setSiteSettings }) => {
          const [settings, setSettings] = useState(siteSettings);
          const [saved, setSaved] = useState(false);

          useEffect(() => setSettings(siteSettings), [siteSettings]);

          const handleChange = (e) => {
              const { name, value } = e.target;
              setSettings(prev => ({ ...prev, [name]: name === 'defaultDiscount' ? parseFloat(value) || 0 : value }));
          };

          const handleSubmit = (e) => {
              e.preventDefault();
              setSiteSettings(settings);
              setSaved(true);
              setTimeout(() => setSaved(false), 2000);
          };

          const QR_CODE_URL = `https://api.qrserver.com/v1/create-qr-code/?size=150x150&data=upi://pay?pa=${settings.upiId}&pn=CheapWeb Store&cu=INR&bgcolor=0f172a&color=f8fafc`;

          return (
              <div>
                  <h2 className="text-2xl font-semibold mb-4 text-text-primary">Site Settings</h2>
                  <form onSubmit={handleSubmit} className="space-y-4 max-w-lg">
                      <div>
                          <label htmlFor="upiId" className="block text-text-secondary text-sm font-bold mb-2">UPI ID</label>
                          <input type="text" id="upiId" name="upiId" value={settings.upiId} onChange={handleChange} placeholder="your-upi-id@bank" className="w-full p-2 bg-background rounded border border-border-color focus:ring-primary focus:border-primary" required />
                      </div>
                      <div>
                          <label htmlFor="defaultDiscount" className="block text-text-secondary text-sm font-bold mb-2">Default Discount (%)</label>
                          <input type="number" id="defaultDiscount" name="defaultDiscount" min="0" max="99" value={settings.defaultDiscount} onChange={handleChange} placeholder="e.g., 33" className="w-full p-2 bg-background rounded border border-border-color" required />
                      </div>
                      <div className="flex items-center space-x-4">
                          <button type="submit" className="bg-primary hover:bg-primary-hover text-white font-bold py-2 px-4 rounded">Save Settings</button>
                          {saved && <span className="text-success">Settings saved!</span>}
                      </div>
                  </form>
                  <div className="mt-6">
                      <h3 className="font-semibold mb-2 text-text-secondary">Current QR Code Preview</h3>
                      {settings.upiId ? <img src={QR_CODE_URL} alt="UPI QR Code Preview" className="rounded-lg" /> : <p className="text-text-secondary">Enter a UPI ID to see the QR code.</p>}
                  </div>
              </div>
          );
      };

      const AdminDashboard = (props) => {
          const [activeTab, setActiveTab] = useState('orders');

          const getTabClass = (tabName) => {
              return `px-4 py-2 font-semibold rounded-md transition-colors duration-200 flex items-center gap-2 ${
                  activeTab === tabName ? 'bg-primary text-white' : 'bg-surface-accent hover:bg-surface text-text-secondary'
              }`;
          };
          
          const pendingOrderCount = props.pendingOrders.filter(o => o.status === 'pending').length;

          return (
              <div className="space-y-8 animate-fade-in-up">
                  <h1 className="text-3xl font-bold">Admin Dashboard</h1>

                  <div className="flex flex-wrap gap-2 border-b border-border-color pb-4">
                      <button className={getTabClass('orders')} onClick={() => setActiveTab('orders')}>
                          Pending Orders
                          {pendingOrderCount > 0 && <span className="ml-2 bg-danger text-white text-xs font-bold rounded-full px-2 py-1">{pendingOrderCount}</span>}
                      </button>
                      <button className={getTabClass('products')} onClick={() => setActiveTab('products')}>Manage Products</button>
                      <button className={getTabClass('advertisement')} onClick={() => setActiveTab('advertisement')}>Manage Ad</button>
                      <button className={getTabClass('settings')} onClick={() => setActiveTab('settings')}>Settings</button>
                  </div>

                  <div className="bg-surface p-6 rounded-lg shadow-md border border-border-color">
                      {activeTab === 'orders' && <OrderManagement pendingOrders={props.pendingOrders} setPendingOrders={props.setPendingOrders} setAllUserProfiles={props.setAllUserProfiles} />}
                      {activeTab === 'products' && <ProductManagement products={props.products} setProducts={props.setProducts} />}
                      {activeTab === 'advertisement' && <AdManagement ad={props.ad} setAd={props.setAd} />}
                      {activeTab === 'settings' && <SettingsManagement siteSettings={props.siteSettings} setSiteSettings={props.setSiteSettings} />}
                  </div>
              </div>
          );
      };

      // --- MAIN APP COMPONENT ---
      function App() {
          const [currentView, setCurrentView] = useState(View.Home);
          const [products, setProducts] = useLocalStorage('cw-ts-products', MOCK_PRODUCTS);
          const [ad, setAd] = useLocalStorage('cw-ts-ad', MOCK_AD);
          const [allUserProfiles, setAllUserProfiles] = useLocalStorage('cw-ts-userProfiles', {});
          const [pendingOrders, setPendingOrders] = useLocalStorage('cw-ts-pendingOrders', []);
          const [siteSettings, setSiteSettings] = useLocalStorage('cw-ts-siteSettings', INITIAL_SITE_SETTINGS);
          const [selectedProduct, setSelectedProduct] = useState(null);
          const [user, setUser] = useState(null);
          const [isLoading, setIsLoading] = useState(true);

          const firebaseConfig = {
              apiKey: "AIzaSyBxap8Yz-OI-RH7imUVY5y8wTy5dF77cr0",
              authDomain: "cheapwebstore-21cc6.firebaseapp.com",
              projectId: "cheapwebstore-21cc6",
              storageBucket: "cheapwebstore-21cc6.appspot.com",
              messagingSenderId: "1088291811951",
              appId: "1:1088291811951:web:4bea5042bf23b49818139a"
          };

          if (!firebase.apps.length) {
              firebase.initializeApp(firebaseConfig);
          }

          useEffect(() => {
              const unsubscribe = firebase.auth().onAuthStateChanged((currentUser) => {
                  setUser(currentUser);
                  if (!currentUser && currentView === View.AdminDashboard) {
                      navigateTo(View.Home);
                  }
                  setIsLoading(false);
              });
              return () => unsubscribe();
          }, [currentView]);

          const navigateTo = (view) => {
              window.scrollTo(0, 0);
              setCurrentView(view);
          };
          
          const handleGoHome = () => navigateTo(View.Home);

          const handleBuyNow = useCallback((product) => {
              if (!user) {
                  alert("Please log in to make a purchase.");
                  navigateTo(View.Login);
                  return;
              }
              setSelectedProduct(product);
              navigateTo(View.Checkout);
          }, [user]);
          
          const handleRequestPurchase = useCallback((orderData) => {
              const newOrder = {
                  ...orderData,
                  id: Date.now(),
                  status: 'pending',
                  date: new Date().toISOString(),
              };
              setPendingOrders(prev => [...prev, newOrder]);
          }, [setPendingOrders]);

          const handleViewDemo = useCallback((productId, reset = false) => {
              if (!user) return;
              setAllUserProfiles(prev => {
                  const userProfile = prev[user.uid] || { purchasedItems: [], demoViews: {} };
                  const currentViews = userProfile.demoViews[productId] || 0;
                  const updatedDemoViews = {
                      ...userProfile.demoViews,
                      [productId]: reset ? 0 : currentViews + 1,
                  };
                  return { ...prev, [user.uid]: { ...userProfile, demoViews: updatedDemoViews } };
              });
          }, [user, setAllUserProfiles]);

          const handleLogout = useCallback(async () => {
              await firebase.auth().signOut();
              navigateTo(View.Home);
          }, []);
          
          const isAdmin = user?.email === ADMIN_EMAIL;
          const currentUserProfile = (user && allUserProfiles[user.uid]) || { purchasedItems: [], demoViews: {} };
          const currentUserPendingItems = useMemo(() => {
              if (!user) return [];
              return pendingOrders
                  .filter(order => order.userId === user.uid && order.status === 'pending')
                  .map(order => order.productId);
          }, [pendingOrders, user]);
          
          const PrivacyPolicyContent = () => (
              <>
                  <p>Your privacy is important to us. It is CheapWeb Store's policy to respect your privacy regarding any information we may collect from you across our website.</p>
                  <p>We only ask for personal information when we truly need it to provide a service to you. We collect it by fair and lawful means, with your knowledge and consent. We also let you know why we’re collecting it and how it will be used.</p>
                  <p>We only retain collected information for as long as necessary to provide you with your requested service. What data we store, we’ll protect within commercially acceptable means to prevent loss and theft, as well as unauthorized access, disclosure, copying, use or modification.</p>
              </>
          );

          const TermsOfServiceContent = () => (
              <>
                  <p>By accessing the website at CheapWeb Store, you are agreeing to be bound by these terms of service, all applicable laws and regulations, and agree that you are responsible for compliance with any applicable local laws.</p>
                  <p>Permission is granted to temporarily download one copy of the materials (information or software) on CheapWeb Store's website for personal, non-commercial transitory viewing only. This is the grant of a license, not a transfer of title.</p>
              </>
          );

          const ContactSupportContent = () => (
              <>
                  <p>If you have any questions, issues, or require assistance, please do not hesitate to reach out to our support team.</p>
                  <p>You can contact us via email for any inquiries:</p>
                  <p><strong>Email:</strong> <a href="mailto:support@cheapwebstore.com" className="text-primary hover:underline">support@cheapwebstore.com</a></p>
              </>
          );

          const renderView = () => {
              if (isLoading) {
                  return (
                      <div className="flex items-center justify-center h-[60vh]">
                          <div className="w-16 h-16 border-4 border-dashed rounded-full animate-spin border-primary"></div>
                      </div>
                  );
              }
              switch (currentView) {
                  case View.Home:
                      return <HomePage products={products} ad={ad} onBuyNow={handleBuyNow} onViewDemo={handleViewDemo} demoViews={currentUserProfile.demoViews} purchasedItems={currentUserProfile.purchasedItems} pendingItems={currentUserPendingItems} siteSettings={siteSettings} />;
                  case View.Checkout:
                      return selectedProduct && user ? <CheckoutPage product={selectedProduct} user={user} siteSettings={siteSettings} onBack={handleGoHome} onRequestPurchase={handleRequestPurchase} /> : <HomePage products={products} ad={ad} onBuyNow={handleBuyNow} onViewDemo={handleViewDemo} demoViews={currentUserProfile.demoViews} purchasedItems={currentUserProfile.purchasedItems} pendingItems={currentUserPendingItems} siteSettings={siteSettings} />;
                  case View.Login:
                      return <LoginPage navigateTo={navigateTo} />;
                  case View.Register:
                      return <RegisterPage navigateTo={navigateTo} />;
                  case View.AdminDashboard:
                      return isAdmin ? <AdminDashboard products={products} setProducts={setProducts} ad={ad} setAd={setAd} pendingOrders={pendingOrders} setPendingOrders={setPendingOrders} setAllUserProfiles={setAllUserProfiles} siteSettings={siteSettings} setSiteSettings={setSiteSettings} /> : <LoginPage navigateTo={navigateTo} />;
                  case View.Library:
                      return user ? <LibraryPage allProducts={products} purchasedIds={currentUserProfile.purchasedItems} onBack={handleGoHome} /> : <LoginPage navigateTo={navigateTo} />;
                  case View.PrivacyPolicy:
                      return <PolicyPage title="Privacy Policy" onBack={handleGoHome}><PrivacyPolicyContent/></PolicyPage>;
                  case View.TermsOfService:
                      return <PolicyPage title="Terms of Service" onBack={handleGoHome}><TermsOfServiceContent/></PolicyPage>;
                  case View.ContactSupport:
                      return <PolicyPage title="Contact Support" onBack={handleGoHome}><ContactSupportContent/></PolicyPage>;
                  default:
                    return <HomePage products={products} ad={ad} onBuyNow={handleBuyNow} onViewDemo={handleViewDemo} demoViews={currentUserProfile.demoViews} purchasedItems={currentUserProfile.purchasedItems} pendingItems={currentUserPendingItems} siteSettings={siteSettings} />;
              }
          };
          
          return (
              <div className="min-h-screen flex flex-col bg-background relative">
                  <div className="absolute top-0 left-0 w-full h-full bg-[radial-gradient(circle_at_top,_rgba(14,165,233,0.15),_transparent_40%)] -z-10"></div>
                  <Header navigateTo={navigateTo} user={user} onLogout={handleLogout} />
                  <main className="flex-grow container mx-auto px-4 sm:px-6 lg:px-8 py-12 sm:py-16">
                      {renderView()}
                  </main>
                  <Footer navigateTo={navigateTo}/>
              </div>
          );
      }

      const rootElement = document.getElementById('root');
      if (!rootElement) {
        throw new Error("Could not find root element to mount to");
      }
      const root = ReactDOM.createRoot(rootElement);
      root.render(
        <React.StrictMode>
          <App />
        </React.StrictMode>
      );
    </script>
  </body>
</html>
