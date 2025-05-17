import { useState, useEffect } from "react";

const DomainGenius = () => {
  // State for domain input and results
  const [keyword, setKeyword] = useState("");
  const [selectedTld, setSelectedTld] = useState(".com");
  const [isSearching, setIsSearching] = useState(false);
  const [results, setResults] = useState([]);
  const [errorMessage, setErrorMessage] = useState("");
  const [showBanner, setShowBanner] = useState(true);
  const [selectedRegistrar, setSelectedRegistrar] = useState("both"); // "namecheap", "godaddy", or "both"

  // Available TLDs
  const tlds = [".com", ".io", ".ai", ".co", ".net", ".org", ".dev"];
  
  // Registrar pricing data (simulated)
  const registrarPricing = {
    namecheap: {
      ".com": { firstYear: 8.88, renewal: 12.98 },
      ".io": { firstYear: 32.88, renewal: 44.88 },
      ".ai": { firstYear: 79.88, renewal: 89.88 },
      ".co": { firstYear: 24.98, renewal: 29.98 },
      ".net": { firstYear: 12.98, renewal: 15.98 },
      ".org": { firstYear: 11.98, renewal: 14.98 },
      ".dev": { firstYear: 14.98, renewal: 19.98 }
    },
    godaddy: {
      ".com": { firstYear: 11.99, renewal: 19.99 },
      ".io": { firstYear: 39.99, renewal: 49.99 },
      ".ai": { firstYear: 99.99, renewal: 99.99 },
      ".co": { firstYear: 29.99, renewal: 32.99 },
      ".net": { firstYear: 14.99, renewal: 17.99 },
      ".org": { firstYear: 12.99, renewal: 16.99 },
      ".dev": { firstYear: 15.99, renewal: 21.99 }
    }
  };
  
  // Mock API call that simulates checking domain availability across registrars
  const checkDomainAvailability = async (domain) => {
    // Simulate API call delay
    await new Promise(resolve => setTimeout(resolve, Math.random() * 600 + 400));
    
    // Extract TLD from domain
    const tld = domain.substring(domain.lastIndexOf("."));
    
    // Simple mock algorithm to determine availability (in production, use a real API)
    const randomAvailable = Math.random() > 0.4;
    const score = Math.floor(Math.random() * 40) + 60; // Score between 60-100
    
    // Prepare registrar data
    const namecheapPricing = registrarPricing.namecheap[tld] || { firstYear: 14.98, renewal: 17.98 };
    const godaddyPricing = registrarPricing.godaddy[tld] || { firstYear: 15.99, renewal: 18.99 };
    
    // Simulate slight availability differences between registrars
    const availableOnNamecheap = randomAvailable;
    const availableOnGodaddy = Math.random() < 0.9 ? randomAvailable : !randomAvailable;
    
    return {
      domain,
      available: randomAvailable,
      score,
      tld,
      registrars: {
        namecheap: {
          available: availableOnNamecheap,
          firstYearPrice: namecheapPricing.firstYear,
          renewalPrice: namecheapPricing.renewal,
          link: `https://www.namecheap.com/domains/registration/results/?domain=${domain}`
        },
        godaddy: {
          available: availableOnGodaddy,
          firstYearPrice: godaddyPricing.firstYear,
          renewalPrice: godaddyPricing.renewal,
          link: `https://www.godaddy.com/domainsearch/find?domainToCheck=${domain}`
        }
      },
      cheapestPrice: availableOnNamecheap && availableOnGodaddy ? 
        Math.min(namecheapPricing.firstYear, godaddyPricing.firstYear) : 
        availableOnNamecheap ? namecheapPricing.firstYear : 
        availableOnGodaddy ? godaddyPricing.firstYear : null
    };
  };

  // Generate domain suggestions based on keyword
  const generateDomainSuggestions = async () => {
    if (!keyword.trim()) {
      setErrorMessage("Please enter a keyword or business idea");
      return;
    }
    
    setIsSearching(true);
    setErrorMessage("");
    setResults([]);
    
    try {
      // Basic suggestions algorithms
      const suggestions = [
        keyword + selectedTld,
        "get" + keyword + selectedTld,
        "the" + keyword + selectedTld,
        keyword + "app" + selectedTld,
        keyword + "hq" + selectedTld,
        "my" + keyword + selectedTld,
        keyword + "pro" + selectedTld,
        keyword + "hub" + selectedTld,
      ];
      
      // Check availability for each suggestion
      const resultsPromises = suggestions.map(domain => checkDomainAvailability(domain));
      const domainResults = await Promise.all(resultsPromises);
      
      // Sort results by score (descending)
      domainResults.sort((a, b) => b.score - a.score);
      
      setResults(domainResults);
    } catch (error) {
      setErrorMessage("Error searching domains. Please try again.");
      console.error("Domain search error:", error);
    } finally {
      setIsSearching(false);
    }
  };

  // Toggle selected TLD
  const handleTldSelect = (tld) => {
    setSelectedTld(tld);
  };

  return (
    <div className="min-h-screen flex flex-col">
      {/* Announcement Banner */}
      {showBanner && (
        <div className="bg-indigo-600 text-white py-2">
          <div className="container mx-auto px-4 flex justify-between items-center">
            <div></div>
            <p className="text-sm">
              🎉 New: Free Domain API Now Available! <a href="#" className="underline">Learn More</a>
            </p>
            <button 
              onClick={() => setShowBanner(false)} 
              className="text-white text-xl font-bold"
              aria-label="Close announcement"
            >
              ×
            </button>
          </div>
        </div>
      )}

      {/* Header */}
      <header className="bg-white shadow-sm sticky top-0 z-10">
        <div className="container mx-auto px-4">
          <div className="flex justify-between items-center py-4">
            <a href="#" className="text-xl font-bold text-indigo-600 flex items-center">
              <svg 
                width="24" 
                height="24" 
                viewBox="0 0 24 24" 
                fill="none" 
                stroke="currentColor" 
                className="mr-2"
              >
                <path d="M12 2L2 7L12 12L22 7L12 2Z" fill="currentColor"/>
                <path d="M2 17L12 22L22 17" stroke="currentColor" strokeWidth="2"/>
                <path d="M2 12L12 17L22 12" stroke="currentColor" strokeWidth="2"/>
              </svg>
              DomainGenius
            </a>
            
            <nav className="hidden md:flex space-x-8 items-center">
              <a href="#features" className="text-gray-700 hover:text-indigo-600">Features</a>
              <a href="#how-it-works" className="text-gray-700 hover:text-indigo-600">How It Works</a>
              <a href="#pricing" className="text-gray-700 hover:text-indigo-600">Pricing</a>
              <a href="#api" className="text-gray-700 hover:text-indigo-600">API</a>
              <a href="#" className="bg-indigo-600 text-white py-2 px-4 rounded-md font-medium hover:bg-indigo-700">
                Get Started
              </a>
            </nav>
          </div>
        </div>
      </header>

      {/* Hero Section */}
      <section className="bg-gradient-to-r from-indigo-600 to-purple-600 text-white py-12">
        <div className="container mx-auto px-4">
          <div className="md:flex items-center gap-8">
            <div className="md:w-1/2 mb-8 md:mb-0">
              <h1 className="text-4xl font-bold mb-4">Find Your Perfect Domain with AI Intelligence</h1>
              <p className="text-lg mb-6">
                Generate smart domain suggestions with real-time availability and search data for your business.
                Now with free API access for more accurate results!
              </p>
              <div className="flex flex-wrap gap-4">
                <a href="#domain-search" className="bg-white text-indigo-600 py-3 px-6 rounded-md font-medium hover:bg-gray-100">
                  Try For Free
                </a>
                <a href="#how-it-works" className="border border-white text-white py-3 px-6 rounded-md font-medium hover:bg-white hover:bg-opacity-10">
                  See How It Works
                </a>
              </div>
            </div>
            
            <div className="md:w-1/2" id="domain-search">
              <div className="bg-white rounded-lg p-6 shadow-lg">
                <label htmlFor="domain-input" className="block text-gray-700 text-sm font-medium mb-2">
                  Enter your business or idea
                </label>
                <input 
                  type="text" 
                  id="domain-input" 
                  value={keyword}
                  onChange={(e) => setKeyword(e.target.value)}
                  placeholder="e.g., fitness coaching, tech blog" 
                  className="w-full border border-gray-300 rounded-md px-3 py-2 mb-4 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                />
                
                <div className="flex flex-wrap gap-2 mb-4">
                  {tlds.map(tld => (
                    <span 
                      key={tld}
                      className={`px-3 py-1 rounded-full text-xs font-medium cursor-pointer ${
                        selectedTld === tld 
                          ? "bg-indigo-100 text-indigo-800" 
                          : "bg-gray-100 text-gray-800 hover:bg-gray-200"
                      }`}
                      onClick={() => handleTldSelect(tld)}
                    >
                      {tld}
                    </span>
                  ))}
                </div>
                
                <button 
                  onClick={generateDomainSuggestions}
                  disabled={isSearching}
                  className="w-full bg-indigo-600 text-white py-2 px-4 rounded-md font-medium hover:bg-indigo-700 disabled:opacity-70"
                >
                  {isSearching ? "Searching..." : "Generate Domain Ideas"}
                </button>
                
                {errorMessage && (
                  <p className="text-red-500 mt-2 text-sm">{errorMessage}</p>
                )}
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* Registrar Selection */}
      <section className="py-6 bg-gray-50">
        <div className="container mx-auto px-4">
          <div className="flex flex-wrap justify-center gap-4 mb-6">
            <button 
              onClick={() => setSelectedRegistrar("both")}
              className={`px-4 py-2 rounded-md border ${
                selectedRegistrar === "both" 
                  ? "bg-indigo-600 text-white border-indigo-600" 
                  : "bg-white text-gray-800 border-gray-300 hover:bg-gray-50"
              }`}
            >
              All Registrars
            </button>
            <button 
              onClick={() => setSelectedRegistrar("namecheap")}
              className={`px-4 py-2 rounded-md border ${
                selectedRegistrar === "namecheap" 
                  ? "bg-indigo-600 text-white border-indigo-600" 
                  : "bg-white text-gray-800 border-gray-300 hover:bg-gray-50"
              }`}
            >
              Namecheap Only
            </button>
            <button 
              onClick={() => setSelectedRegistrar("godaddy")}
              className={`px-4 py-2 rounded-md border ${
                selectedRegistrar === "godaddy" 
                  ? "bg-indigo-600 text-white border-indigo-600" 
                  : "bg-white text-gray-800 border-gray-300 hover:bg-gray-50"
              }`}
            >
              GoDaddy Only
            </button>
          </div>
        </div>
      </section>

      {/* Results Section - Only shows when there are results */}
      {results.length > 0 && (
        <section className="py-6 bg-gray-50">
          <div className="container mx-auto px-4">
            <h2 className="text-2xl font-bold mb-6 text-center">Domain Suggestions</h2>
            <div className="bg-white rounded-lg shadow overflow-hidden overflow-x-auto">
              <table className="min-w-full divide-y divide-gray-200">
                <thead className="bg-gray-50">
                  <tr>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                      Domain Name
                    </th>
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                      Score
                    </th>
                    {selectedRegistrar === "both" || selectedRegistrar === "namecheap" ? (
                      <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        Namecheap
                      </th>
                    ) : null}
                    {selectedRegistrar === "both" || selectedRegistrar === "godaddy" ? (
                      <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                        GoDaddy
                      </th>
                    ) : null}
                    <th className="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                      Action
                    </th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {results.map((result, index) => (
                    <tr key={index} className={index % 2 === 0 ? "bg-white" : "bg-gray-50"}>
                      <td className="px-4 py-4 whitespace-nowrap font-medium">{result.domain}</td>
                      <td className="px-4 py-4 whitespace-nowrap">
                        <div className="flex items-center">
                          <div className="w-16 bg-gray-200 rounded-full h-2">
                            <div 
                              className={`h-2 rounded-full ${
                                result.score >= 80 ? "bg-green-500" : 
                                result.score >= 60 ? "bg-yellow-500" : "bg-red-500"
                              }`}
                              style={{ width: `${result.score}%` }}
                            ></div>
                          </div>
                          <span className="ml-2 text-sm text-gray-600">{result.score}</span>
                        </div>
                      </td>
                      
                      {/* Namecheap Column */}
                      {(selectedRegistrar === "both" || selectedRegistrar === "namecheap") && (
                        <td className="px-4 py-4 whitespace-nowrap">
                          {result.registrars.namecheap.available ? (
                            <div>
                              <span className="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800 mb-1">
                                Available
                              </span>
                              <div className="text-sm">
                                <div className="font-medium">${result.registrars.namecheap.firstYearPrice}</div>
                                <div className="text-gray-500 text-xs">Renewal: ${result.registrars.namecheap.renewalPrice}/yr</div>
                              </div>
                            </div>
                          ) : (
                            <span className="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">
                              Taken
                            </span>
                          )}
                        </td>
                      )}
                      
                      {/* GoDaddy Column */}
                      {(selectedRegistrar === "both" || selectedRegistrar === "godaddy") && (
                        <td className="px-4 py-4 whitespace-nowrap">
                          {result.registrars.godaddy.available ? (
                            <div>
                              <span className="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800 mb-1">
                                Available
                              </span>
                              <div className="text-sm">
                                <div className="font-medium">${result.registrars.godaddy.firstYearPrice}</div>
                                <div className="text-gray-500 text-xs">Renewal: ${result.registrars.godaddy.renewalPrice}/yr</div>
                              </div>
                            </div>
                          ) : (
                            <span className="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">
                              Taken
                            </span>
                          )}
                        </td>
                      )}
                      
                      {/* Action Column */}
                      <td className="px-4 py-4 whitespace-nowrap">
                        {(result.registrars.namecheap.available || result.registrars.godaddy.available) ? (
                          <div className="flex flex-col space-y-2">
                            {result.registrars.namecheap.available && (selectedRegistrar === "both" || selectedRegistrar === "namecheap") && (
                              <a 
                                href={result.registrars.namecheap.link} 
                                target="_blank" 
                                rel="noopener noreferrer"
                                className="bg-blue-600 text-white py-1 px-3 rounded text-xs hover:bg-blue-700 flex items-center justify-center"
                              >
                                Buy at Namecheap
                              </a>
                            )}
                            {result.registrars.godaddy.available && (selectedRegistrar === "both" || selectedRegistrar === "godaddy") && (
                              <a 
                                href={result.registrars.godaddy.link} 
                                target="_blank" 
                                rel="noopener noreferrer"
                                className="bg-green-600 text-white py-1 px-3 rounded text-xs hover:bg-green-700 flex items-center justify-center"
                              >
                                Buy at GoDaddy
                              </a>
                            )}
                          </div>
                        ) : (
                          <button className="bg-gray-200 text-gray-700 py-1 px-3 rounded text-sm">
                            Make Offer
                          </button>
                        )}
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
            
            {/* Price comparison legend */}
            <div className="mt-4 bg-white p-4 rounded-lg shadow">
              <h3 className="font-medium text-lg mb-2">Why prices differ?</h3>
              <p className="text-sm text-gray-600 mb-2">
                Domain registrars often have different pricing strategies:
              </p>
              <ul className="text-sm text-gray-600 list-disc pl-5 space-y-1">
                <li>Namecheap typically offers lower first-year prices but might have higher renewal rates</li>
                <li>GoDaddy sometimes includes extra features like privacy protection in their pricing</li>
                <li>Special promotions can significantly affect pricing for specific domains</li>
                <li>We recommend comparing total cost over multiple years before purchasing</li>
              </ul>
            </div>
          </div>
        </section>
      )}

      {/* API Section */}
      <section className="py-12 bg-white" id="api">
        <div className="container mx-auto px-4">
          <h2 className="text-2xl font-bold mb-6 text-center">Free Domain API</h2>
          <div className="md:flex items-start gap-8">
            <div className="md:w-1/2 mb-8 md:mb-0">
              <div className="bg-gray-800 text-white p-6 rounded-lg shadow-lg">
                <h3 className="text-lg font-medium mb-4">API Endpoint</h3>
                <div className="bg-gray-900 p-4 rounded-md mb-6 font-mono text-sm overflow-x-auto">
                  GET https://api.domaingenius.com/v1/check?domain=yourname.com&registrars=all
                </div>
                
                <h3 className="text-lg font-medium mb-4">Response Format</h3>
                <div className="bg-gray-900 p-4 rounded-md font-mono text-sm overflow-x-auto">
{`{
  "domain": "yourname.com",
  "available": true,
  "score": 85,
  "registrars": {
    "namecheap": {
      "available": true,
      "firstYearPrice": 8.88,
      "renewalPrice": 12.98,
      "link": "https://www.namecheap.com/domains/registration/results/?domain=yourname.com"
    },
    "godaddy": {
      "available": true,
      "firstYearPrice": 11.99,
      "renewalPrice": 19.99,
      "link": "https://www.godaddy.com/domainsearch/find?domainToCheck=yourname.com"
    }
  },
  "suggestions": [
    {"domain": "yourname.io", "available": true},
    {"domain": "yourname.co", "available": false}
  ]
}`}
                </div>
              </div>
            </div>
            
            <div className="md:w-1/2">
              <h3 className="text-xl font-bold mb-4">Free API Access</h3>
              <p className="mb-4">
                Our free API gives you access to real-time domain availability and pricing data from Namecheap and GoDaddy with up to 50 requests per day.
              </p>
              
              <h4 className="text-lg font-medium mb-2">API Features:</h4>
              <ul className="list-disc pl-5 mb-6 space-y-2">
                <li>Real-time domain availability checking across major registrars</li>
                <li>Current pricing from Namecheap and GoDaddy</li>
                <li>Renewal pricing information to avoid surprises</li>
                <li>Domain quality scoring algorithm</li>
                <li>Direct registration links to registrars</li>
                <li>Simple REST API with JSON responses</li>
                <li>Cross-origin support for browser applications</li>
                <li>Free tier with 50 daily requests</li>
              </ul>
              
              <div className="bg-gray-100 p-4 rounded-lg">
                <h4 className="font-medium mb-2">Get Your API Key</h4>
                <div className="flex">
                  <input
                    type="email"
                    placeholder="Enter your email"
                    className="flex-grow border border-gray-300 rounded-l-md px-3 py-2 focus:outline-none focus:ring-2 focus:ring-indigo-500"
                  />
                  <button className="bg-indigo-600 text-white px-4 py-2 rounded-r-md hover:bg-indigo-700">
                    Get Key
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </section>

      {/* Footer */}
      <footer className="bg-gray-800 text-gray-200 py-8 mt-auto">
        <div className="container mx-auto px-4">
          <div className="grid grid-cols-1 md:grid-cols-3 gap-8">
            <div>
              <h3 className="text-white text-lg font-bold mb-4">DomainGenius</h3>
              <p className="mb-4">Find the perfect domain name with AI-powered intelligence and free API access.</p>
            </div>
            
            <div>
              <h4 className="text-white font-medium mb-4">Quick Links</h4>
              <ul className="space-y-2">
                <li><a href="#features" className="hover:text-white">Features</a></li>
                <li><a href="#pricing" className="hover:text-white">Pricing</a></li>
                <li><a href="#api" className="hover:text-white">API</a></li>
                <li><a href="#" className="hover:text-white">Contact</a></li>
              </ul>
            </div>
            
            <div>
              <h4 className="text-white font-medium mb-4">Connect</h4>
              <div className="flex space-x-4">
                <a href="#" aria-label="Twitter" className="text-xl">🐦</a>
                <a href="#" aria-label="Facebook" className="text-xl">📘</a>
                <a href="#" aria-label="Instagram" className="text-xl">📷</a>
                <a href="#" aria-label="LinkedIn" className="text-xl">💼</a>
              </div>
            </div>
          </div>
          
          <div className="mt-8 pt-4 border-t border-gray-700 text-center">
            <p>&copy; 2025 DomainGenius. All rights reserved.</p>
          </div>
        </div>
      </footer>
    </div>
  );
};

export default DomainGenius;
